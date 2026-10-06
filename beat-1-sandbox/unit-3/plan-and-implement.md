# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

jordannLai

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/32#issuecomment-6025402204

Text of the comment as posted (copied from the comment on GitHub, unedited):

Here's my plan for fixing this, based on the reproduction I posted above. In that run, Postgres rows went 1 → 0 after `DELETE`, but the profile's Chroma chunks stayed at 1 → 1 and were still returned by `VectorStore.query` and `HybridRetriever` from a fresh process.

**Cause:** `delete_profile` (`core/services/profile_service.py`) only does ORM deletes, and `VectorStore` has no profile-scoped delete. It only has `delete_by_source_id`.

**Proposed change (one bounded fix):**

1. `rag/retriever/vector_store.py`: add `delete_profile_chunks(profile_id)`. It drops the `profile_{profile_id}` collection, which is the one `HybridRetriever` reads. When that collection doesn't exist, it does nothing and returns 0. That case matters: chromadb 1.5.9 raises `NotFoundError` there, and since no route ingests anything today, most profiles have no collection.
2. `core/services/profile_service.py`: `delete_profile` calls it after the ORM deletes and **before** `db.commit()`. If the cleanup fails, the existing handler rolls back and re-raises, so we never end up in the reported state (rows gone, chunks left). The route itself doesn't change.

**Not in scope:** wiring `IngestionPipeline` into a route, switching `VectorStore` to the `vector_db_url` server in `core/config.py`, or refactoring the collection-naming in `hybrid.py`. I'd file the config mismatch separately if that's useful.

**How I'll show it's fixed:**

- I'll re-run my repro driver. Expected: `DELETE` → 204, chunks after delete **0** (was 1), both retrieval hit counts **0** (were 1), and `ISSUE #32 REPRODUCED: False`.
- New unit tests in `tests/unit/test_vector_store.py` will cover: the collection is removed, other profiles are untouched, and a missing collection is a no-op.
- New unit tests in `tests/unit/test_profile_service.py` will cover: the cleanup runs before commit, a cleanup failure rolls back without committing, and a not-found profile never touches the store.
- `make check` and `make test-unit` will be green.

**Open questions I'd like input on:**

- This puts a `rag` import into `core/services` for the first time. If you'd rather keep `core` independent of `rag`, I can make the same call from the route handler instead.
- I'm assuming a profile's chunks only live in its own `profile_<id>` collection, because that's the only one retrieval reads and where my repro stored them. If chunks might go into a shared collection, I'd add a `where={"profile_id": ...}` delete for that too.
- The two stores can't be updated atomically. If the Chroma delete succeeds but `db.commit()` then fails, the profile survives without its chunks. I picked that over leaving orphaned chunks because chunks can be re-ingested, but I'm open to the other ordering.

I've only verified this on macOS arm64 with chromadb 1.5.9 (local `.chromadb`) and the mock embedding provider. I haven't tested it against the dockerized Chroma server.

I'll do the work on `fix/32-vector-store-cleanup` in my fork.

AI disclosure: I used Claude Code to help investigate the code and draft this plan. I reviewed and verified the diagnosis, file references, and Chroma behavior myself against my own reproduction.

---

## Your branch

**Branch**

`fix/32-vector-store-cleanup`, on my fork: https://github.com/jordannLai/pathreview-ai301-fa26-s3/tree/fix/32-vector-store-cleanup

The change is one commit, `83f00afbf79b285824878d5a5a4add068e00a621`
(`fix(api): delete a profile's vector-store chunks on profile deletion`, `Fixes #32`). It
touches four files: `rag/retriever/vector_store.py`, `core/services/profile_service.py`,
`tests/unit/test_vector_store.py`, `tests/unit/test_profile_service.py`.

**Evidence**

I re-ran my Unit 2 driver (`repro32.py`, the same script that is in my repro comment on #32)
unchanged, twice on 2026-10-06. The first run used the original deletion code and the
second used the fix. Each run started with an empty `.chromadb`. The environment was the
same as in Unit 2: macOS arm64, Python 3.14.7, `postgres:16-alpine` on host port 5434,
Alembic `002 (head)`, chromadb 1.5.9 local `PersistentClient`, `LLM_PROVIDER=mock`.

After the driver, each run also ran a fresh-process probe, `strict_probe.py`. It opens the
store with `client.get_collection`, which never creates a collection, counts chunks tagged
with the profile's `profile_id` across every collection, and runs a semantic query worded
without the sentinel token.

Commands, run from the repo root:

```bash
docker compose up -d db                       # Postgres on :5434, healthy
.venv/bin/alembic current                     # 002 (head)

# BEFORE: the two source files restored to the original code at 2f4e82f
git show 2f4e82f:rag/retriever/vector_store.py    > rag/retriever/vector_store.py
git show 2f4e82f:core/services/profile_service.py > core/services/profile_service.py
rm -rf .chromadb
.venv/bin/uvicorn api.main:app --host 127.0.0.1 --port 8000 &
PYTHONPATH=. .venv/bin/python repro32.py
PYTHONPATH=. .venv/bin/python strict_probe.py <profile_id> <marker>
kill %1

# AFTER: the fixed files (as committed in 83f00af) put back, same commands
rm -rf .chromadb
.venv/bin/uvicorn api.main:app --host 127.0.0.1 --port 8000 &
PYTHONPATH=. .venv/bin/python repro32.py
PYTHONPATH=. .venv/bin/python strict_probe.py <profile_id> <marker>
kill %1
```

`strict_probe.py`:

```python
"""Fresh-process, non-creating probe of the vector store for one profile id."""
import sys
from chromadb.errors import NotFoundError
from rag.retriever.vector_store import VectorStore
from ingestion.embeddings.provider import MockEmbeddingProvider
pid, marker = sys.argv[1], sys.argv[2]
vs = VectorStore(persist_dir=".chromadb")
names = [c.name if hasattr(c, "name") else c for c in vs.client.list_collections()]
print("collections on disk:", names)
total = sum(len(vs.client.get_collection(name=n).get(where={"profile_id": pid})["ids"]) for n in names)
print(f"chunks tagged profile_id={pid} across ALL collections: {total}")
try:
    col = vs.client.get_collection(name=f"profile_{pid}")
    res = col.query(query_embeddings=[MockEmbeddingProvider().embed(["widget pipeline built in Python"])[0]], n_results=1)
    hits = len(res["ids"][0])
    print(f"semantic query (non-marker wording) -> {hits} hits" + (f"; marker in text: {marker in res['documents'][0][0]}" if hits else ""))
except NotFoundError as e:
    print(f"client.get_collection('profile_{pid}') -> NotFoundError: {e}")
```

Summary of the two runs (the full output is pasted below):

| Measurement | Before (original code) | After (fix) |
|---|---|---|
| `DELETE /profiles/{id}` | 204, then `GET` → 404 | 204, then `GET` → 404 |
| Postgres `profiles` / `ingested_sources` / `reviews` rows after delete | 0 / 0 / 0 | 0 / 0 / 0 |
| `collections present` right after delete | `['profile_e24db6ca-…']` | `[]` |
| Chroma chunks with the profile's `profile_id`, before → after | 1 → **1** | 1 → **0** |
| `VectorStore.query` hits, before → after | 1 → **1** | 1 → **0** |
| `HybridRetriever.retrieve` hits, before → after | 1 → **1** | 1 → **0** |
| Fresh-process probe: tagged chunks across all collections | 1 | 0 |
| `ISSUE #32 REPRODUCED` | **True** | **False** |

In the after run, the probe still lists one `profile_<id>` collection on disk. It is
empty: the driver's own step-7 check calls `VectorStore.get_collection`, which recreates
the collection after the delete. The `collections present: []` line prints before that
call. The probe confirms the recreated collection holds 0 chunks and returns 0 hits.

The API server log for the after run shows the cleanup running inside the delete request,
before the commit (`profile_deleted_cascade` is logged right after `db.commit()`):

```
2026-10-06 17:07:35 [info     ] deleted_profile_chunks         collection=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a count=1 profile_id=3914fdf2-294c-4320-a3af-edb5e0cb7e5a request_id=5e2c9a81-4bf9-4e3f-ab96-1f6a6731f75f
2026-10-06 17:07:35 [info     ] profile_deleted_cascade        profile_id=3914fdf2-294c-4320-a3af-edb5e0cb7e5a request_id=5e2c9a81-4bf9-4e3f-ab96-1f6a6731f75f
INFO:     127.0.0.1:59699 - "DELETE /profiles/3914fdf2-294c-4320-a3af-edb5e0cb7e5a HTTP/1.1" 204 No Content
```

**Before: original code, full output**

```
marker      : PATHREVIEW-REPRO-ISSUE32-755CEA37
test user   : repro-issue32-8e3ac108@example.com

======================================================================
STEP 1  Create disposable user + authenticate
======================================================================
POST /auth/register -> 200
POST /auth/login    -> 200

======================================================================
STEP 2  Create profile via POST /profiles
======================================================================
POST /profiles      -> 200
{
  "id": "e24db6ca-487b-4ce6-83cc-1aa750cda640",
  "user_id": "2f6435a9-d9ac-4b0b-87ff-9c84d94baf89",
  "github_username": "repro-pathreview-repro-issue32-755cea37",
  "portfolio_url": "https://example.com/PATHREVIEW-REPRO-ISSUE32-755CEA37",
  "created_at": "2026-10-07T01:07:22.382514Z",
  "resume_filename": "disposable_resume.md"
}

PROFILE ID: e24db6ca-487b-4ce6-83cc-1aa750cda640

======================================================================
STEP 3  Ingest resume via IngestionPipeline -> local .chromadb
======================================================================
2026-10-06 17:07:23 [info     ] created_collection             collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] Starting resume ingestion      filename=disposable_resume.md profile_id=e24db6ca-487b-4ce6-83cc-1aa750cda640 source_id=resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779
2026-10-06 17:07:27 [warning  ] Could not check if source already ingested error="'NoneType' object has no attribute 'query'" source_id=resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779
2026-10-06 17:07:27 [info     ] Resume parsed successfully     sections=['Experience', 'Skills', 'Summary']
2026-10-06 17:07:27 [info     ] Resume chunked successfully    chunk_count=1
2026-10-06 17:07:27 [info     ] Starting batch embedding processing chunk_count=1
2026-10-06 17:07:27 [info     ] Processing embedding batch     batch_end=1 batch_num=1 batch_start=0 total=1
2026-10-06 17:07:27 [info     ] Generated embeddings for batch embedding_count=1
2026-10-06 17:07:27 [debug    ] Stored embedding in vector DB  embedding_id=resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779_chunk_0
2026-10-06 17:07:27 [info     ] Batch embedding processing complete stored_count=1
2026-10-06 17:07:27 [info     ] Resume embeddings stored       chunk_count=1
2026-10-06 17:07:27 [info     ] Recording ingested source      chunk_count=1 profile_id=e24db6ca-487b-4ce6-83cc-1aa750cda640 source_id=resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779 source_type=resume
ingest_resume -> source_id=resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779 chunk_count=1 skipped=False

======================================================================
STEP 4  BEFORE delete: vector-store state
======================================================================
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
[BEFORE] collection 'profile_e24db6ca-487b-4ce6-83cc-1aa750cda640' exists, total chunks = 1
[BEFORE] chunks with metadata profile_id == e24db6ca-487b-4ce6-83cc-1aa750cda640: 1
[BEFORE] sample chunk id  : resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779_chunk_0
[BEFORE] sample metadata  : {'source_id': 'resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779', 'chunk_index': 0, 'profile_id': 'e24db6ca-487b-4ce6-83cc-1aa750cda640', 'source_type': 'resume'}
[BEFORE] marker in text   : True
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] vector_query_complete          collection=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640 results_count=1
[BEFORE] VectorStore.query  -> 1 hits, top score=0.4983, marker in text=True
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] keyword_index_built            chunk_count=1
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] vector_query_complete          collection=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640 results_count=1
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] keyword_search_complete        query_len=1 results_count=1
2026-10-06 17:07:27 [info     ] hybrid_retrieval_complete      blended_count=1 filtered_count=1 final_count=1 keyword_results=1 query_len=33 vector_results=1
[BEFORE] HybridRetriever    -> 1 hits, top score=0.7000
[BEFORE] postgres profiles rows for id: 1

======================================================================
STEP 5  DELETE /profiles/{profile_id}
======================================================================
DELETE /profiles/e24db6ca-487b-4ce6-83cc-1aa750cda640
  -> HTTP 204
  -> body: '' (len 0)
GET /profiles/e24db6ca-487b-4ce6-83cc-1aa750cda640 after delete -> HTTP 404 {"detail":"Profile not found"}

======================================================================
STEP 6  AFTER delete: PostgreSQL state
======================================================================
profiles          rows for id: 0
ingested_sources  rows for id: 0
reviews           rows for id: 0

======================================================================
STEP 7  AFTER delete: vector-store state (same profile_id, same store)
======================================================================
collections present: ['profile_e24db6ca-487b-4ce6-83cc-1aa750cda640']
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
[AFTER] collection 'profile_e24db6ca-487b-4ce6-83cc-1aa750cda640' exists, total chunks = 1
[AFTER] chunks with metadata profile_id == e24db6ca-487b-4ce6-83cc-1aa750cda640: 1
[AFTER] sample chunk id  : resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779_chunk_0
[AFTER] sample metadata  : {'source_id': 'resume_e24db6ca-487b-4ce6-83cc-1aa750cda640_e5bc6791777ab779', 'source_type': 'resume', 'chunk_index': 0, 'profile_id': 'e24db6ca-487b-4ce6-83cc-1aa750cda640'}
[AFTER] marker in text   : True
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] vector_query_complete          collection=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640 results_count=1
[AFTER] VectorStore.query  -> 1 hits, top score=0.4983, marker in text=True
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] keyword_index_built            chunk_count=1
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] vector_query_complete          collection=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640 results_count=1
2026-10-06 17:07:27 [info     ] retrieved_collection           collection_name=profile_e24db6ca-487b-4ce6-83cc-1aa750cda640
2026-10-06 17:07:27 [info     ] keyword_search_complete        query_len=1 results_count=1
2026-10-06 17:07:27 [info     ] hybrid_retrieval_complete      blended_count=1 filtered_count=1 final_count=1 keyword_results=1 query_len=33 vector_results=1
[AFTER] HybridRetriever    -> 1 hits, top score=0.7000

======================================================================
RESULT
======================================================================
profile_id                      : e24db6ca-487b-4ce6-83cc-1aa750cda640
postgres profile rows  before/after : 1 / 0
chroma chunks (profile_id) before/after : 1 / 1
vector query hits      before/after : 1 / 1
hybrid retrieve hits   before/after : 1 / 1

ISSUE #32 REPRODUCED: True

=== STRICT FRESH-PROCESS PROBE ===
collections on disk: ['profile_e24db6ca-487b-4ce6-83cc-1aa750cda640']
chunks tagged profile_id=e24db6ca-487b-4ce6-83cc-1aa750cda640 across ALL collections: 1
semantic query (non-marker wording) -> 1 hits; marker in text: True
```

**After: fixed code, full output**

```
marker      : PATHREVIEW-REPRO-ISSUE32-149717D3
test user   : repro-issue32-197f864f@example.com

======================================================================
STEP 1  Create disposable user + authenticate
======================================================================
POST /auth/register -> 200
POST /auth/login    -> 200

======================================================================
STEP 2  Create profile via POST /profiles
======================================================================
POST /profiles      -> 200
{
  "id": "3914fdf2-294c-4320-a3af-edb5e0cb7e5a",
  "user_id": "71e261ba-7a1a-476c-be37-21d50f2fa6f8",
  "github_username": "repro-pathreview-repro-issue32-149717d3",
  "portfolio_url": "https://example.com/PATHREVIEW-REPRO-ISSUE32-149717D3",
  "created_at": "2026-10-07T01:07:35.110358Z",
  "resume_filename": "disposable_resume.md"
}

PROFILE ID: 3914fdf2-294c-4320-a3af-edb5e0cb7e5a

======================================================================
STEP 3  Ingest resume via IngestionPipeline -> local .chromadb
======================================================================
2026-10-06 17:07:35 [info     ] created_collection             collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [info     ] Starting resume ingestion      filename=disposable_resume.md profile_id=3914fdf2-294c-4320-a3af-edb5e0cb7e5a source_id=resume_3914fdf2-294c-4320-a3af-edb5e0cb7e5a_85a451f51bfec4e5
2026-10-06 17:07:35 [warning  ] Could not check if source already ingested error="'NoneType' object has no attribute 'query'" source_id=resume_3914fdf2-294c-4320-a3af-edb5e0cb7e5a_85a451f51bfec4e5
2026-10-06 17:07:35 [info     ] Resume parsed successfully     sections=['Experience', 'Skills', 'Summary']
2026-10-06 17:07:35 [info     ] Resume chunked successfully    chunk_count=1
2026-10-06 17:07:35 [info     ] Starting batch embedding processing chunk_count=1
2026-10-06 17:07:35 [info     ] Processing embedding batch     batch_end=1 batch_num=1 batch_start=0 total=1
2026-10-06 17:07:35 [info     ] Generated embeddings for batch embedding_count=1
2026-10-06 17:07:35 [debug    ] Stored embedding in vector DB  embedding_id=resume_3914fdf2-294c-4320-a3af-edb5e0cb7e5a_85a451f51bfec4e5_chunk_0
2026-10-06 17:07:35 [info     ] Batch embedding processing complete stored_count=1
2026-10-06 17:07:35 [info     ] Resume embeddings stored       chunk_count=1
2026-10-06 17:07:35 [info     ] Recording ingested source      chunk_count=1 profile_id=3914fdf2-294c-4320-a3af-edb5e0cb7e5a source_id=resume_3914fdf2-294c-4320-a3af-edb5e0cb7e5a_85a451f51bfec4e5 source_type=resume
ingest_resume -> source_id=resume_3914fdf2-294c-4320-a3af-edb5e0cb7e5a_85a451f51bfec4e5 chunk_count=1 skipped=False

======================================================================
STEP 4  BEFORE delete: vector-store state
======================================================================
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
[BEFORE] collection 'profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a' exists, total chunks = 1
[BEFORE] chunks with metadata profile_id == 3914fdf2-294c-4320-a3af-edb5e0cb7e5a: 1
[BEFORE] sample chunk id  : resume_3914fdf2-294c-4320-a3af-edb5e0cb7e5a_85a451f51bfec4e5_chunk_0
[BEFORE] sample metadata  : {'chunk_index': 0, 'source_type': 'resume', 'source_id': 'resume_3914fdf2-294c-4320-a3af-edb5e0cb7e5a_85a451f51bfec4e5', 'profile_id': '3914fdf2-294c-4320-a3af-edb5e0cb7e5a'}
[BEFORE] marker in text   : True
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [info     ] vector_query_complete          collection=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a results_count=1
[BEFORE] VectorStore.query  -> 1 hits, top score=0.5003, marker in text=True
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [info     ] keyword_index_built            chunk_count=1
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [info     ] vector_query_complete          collection=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a results_count=1
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [info     ] keyword_search_complete        query_len=1 results_count=1
2026-10-06 17:07:35 [info     ] hybrid_retrieval_complete      blended_count=1 filtered_count=1 final_count=1 keyword_results=1 query_len=33 vector_results=1
[BEFORE] HybridRetriever    -> 1 hits, top score=0.7000
[BEFORE] postgres profiles rows for id: 1

======================================================================
STEP 5  DELETE /profiles/{profile_id}
======================================================================
DELETE /profiles/3914fdf2-294c-4320-a3af-edb5e0cb7e5a
  -> HTTP 204
  -> body: '' (len 0)
GET /profiles/3914fdf2-294c-4320-a3af-edb5e0cb7e5a after delete -> HTTP 404 {"detail":"Profile not found"}

======================================================================
STEP 6  AFTER delete: PostgreSQL state
======================================================================
profiles          rows for id: 0
ingested_sources  rows for id: 0
reviews           rows for id: 0

======================================================================
STEP 7  AFTER delete: vector-store state (same profile_id, same store)
======================================================================
collections present: []
2026-10-06 17:07:35 [info     ] created_collection             collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
[AFTER] collection 'profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a' exists, total chunks = 0
[AFTER] chunks with metadata profile_id == 3914fdf2-294c-4320-a3af-edb5e0cb7e5a: 0
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [info     ] vector_query_complete          collection=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a results_count=0
[AFTER] VectorStore.query  -> 0 hits
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [info     ] vector_query_complete          collection=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a results_count=0
2026-10-06 17:07:35 [info     ] retrieved_collection           collection_name=profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a
2026-10-06 17:07:35 [warning  ] keyword_search_empty_index
2026-10-06 17:07:35 [info     ] hybrid_retrieval_complete      blended_count=0 filtered_count=0 final_count=0 keyword_results=0 query_len=33 vector_results=0
[AFTER] HybridRetriever    -> 0 hits

======================================================================
RESULT
======================================================================
profile_id                      : 3914fdf2-294c-4320-a3af-edb5e0cb7e5a
postgres profile rows  before/after : 1 / 0
chroma chunks (profile_id) before/after : 1 / 0
vector query hits      before/after : 1 / 0
hybrid retrieve hits   before/after : 1 / 0

ISSUE #32 REPRODUCED: False

=== STRICT FRESH-PROCESS PROBE ===
collections on disk: ['profile_3914fdf2-294c-4320-a3af-edb5e0cb7e5a']
chunks tagged profile_id=3914fdf2-294c-4320-a3af-edb5e0cb7e5a across ALL collections: 0
semantic query (non-marker wording) -> 0 hits
```

Unit tests on the branch: `make test-unit` gives `381 passed, 53 xfailed`. Without the
change it gives `375 passed, 53 xfailed`, so the difference is my 6 new tests. All 6 fail
when the two source files are reverted to the original code.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

All runs used `run_eval.py` with the installed `~/.claude/skills/plan-check/` rubric,
evidence guide, and procedure, graded by Sonnet. In order:

1. **Run 1, full (plus `--include-calibration`): `agreement: 19/20 scored items  (bar: 18/20: PASS)`.**
   Categories: `clear-accept 6/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`.
   The 4 calibration packages agreed 4/4. The one miss was `pkg-03` (gold accept,
   graded reject, `failed: ai-disclosure`).
2. **Run 2, `--only pkg-03,pkg-20,pkg-04,pkg-09` after revising `ai-disclosure`: 4/4 agreed.**
   This was a partial run, so it printed no bar verdict.
3. **Run 3, full, saved with `--save-run eval-run.txt`: `agreement: 19/20 scored items  (bar: 18/20: PASS)`.**
   Categories: `clear-accept 6/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`.
   `pkg-03` now agreed. The one miss was `pkg-14` (gold accept, graded reject,
   `failed: grounded-cause`).

Final score: **19/20**, matching the agreement line in the committed `eval-run.txt`.

**Package analysis**

**`pkg-14` (zellij-org/zellij#5174).** My rubric decided **reject** in the committed run.
The gold label is **accept** (gold note: "honestly scoped-down: reattach handshake fix with
a regression-window repro; defers the untestable Windows variant and says so; arguable on
the deferral, ready as scoped").

The deciding check was `grounded-cause`. The grader's evidence line was: "Cache control
(after rm -rf ~/.cache/zellij next attach is clean, then leaks again) contradicts a
stdin-wiring-order cause; plan hand-waves it and defers any caching change".

Why my rubric read it that way: the package's repro has two controls. The 0.44.1 control
is clean, and the cache control reads "After `rm -rf ~/.cache/zellij`: the next attach is
clean, the one after leaks again". My `grounded-cause` pass condition says "A control rules
a cause out when it shows the blamed component behaving correctly under the failing
conditions". My procedure then makes the grader write, for every control, "If CAUSE were
true, CONTROL-n would have ___; it actually ___". The plan blames stdin being wired before
the OSC responses are consumed on reattach. A clean reattach after clearing the cache looks,
on a quick read, like the reattach path behaving correctly, so the grader marked the
control `contradicts`. The plan does account for it ("with an empty cache the color data
is refetched along the fresh-attach path once"), which makes that one attach a fresh-attach
path rather than a reattach. My rubric has no rule that a control the plan explicitly
explains does not count against the cause.

The same check, with unchanged wording, passed this package in Run 1 ("the plan explains
the cache control as the one-time fresh-attach path; no control contradicts it"). So this
is a borderline control where my pass condition leaves room for both readings, not a
change I introduced. I agree with the gold label here: the plan's explanation is consistent
with every control.

**Check rationale**

From `tools/plan-check/rubric.md`, exactly as it reads now:

> | ai-disclosure | The repo-facts block's contribution policy / AI policy line (live: the repo's CONTRIBUTING and any AI policy), read against the plan comment's text. Treat every package as AI-assisted work. | First decide where the policy demands disclosure. If the policy states no AI rule, or demands disclosure only somewhere other than issue comments (e.g. "state it in the pull request") and says no disclosure is needed for issue comments, pass. If the policy demands disclosure of all AI usage, or of AI use in issues/comments, pass only if the plan comment itself discloses AI assistance (that AI was used and how/to what extent, plus the tool if the policy asks for it). Fail if that required disclosure is absent from the comment, or if the comment's own text visibly breaks another explicit comment rule in the policy (e.g. it says it was AI-generated where the policy forbids that). "Treat every package as AI-assisted" means the *work* may be AI-assisted; it never means the comment was AI-written. Take the comment's statement about its own authorship ("this comment is in my own words") at face value: that statement complies with an own-words rule, it does not violate it. | required |

Why it reads this way: my first version ended with "Fail if that required disclosure is
absent from the comment, or if the comment breaks another explicit comment rule in the
policy (e.g. the policy forbids AI-written comments and the comment claims or shows
otherwise)." Combined with "Treat every package as AI-assisted work", that sentence made
the grader fail `pkg-03` in Run 1. Ripgrep's policy says "comments to maintainers must be
written by humans in their own words", and the grader read "AI-assisted package" as "the
comment was AI-written". It then treated the candidate's own compliant sentence ("this
comment is in my own words") as a false claim.

I kept the instruction to treat every package as AI-assisted, because that is what makes
`pkg-20` fail: ghostty requires disclosing all AI usage, and that comment has none. I added
two sentences. One says the AI-assisted assumption is about the *work* and never means the
comment was AI-written. The other says to take the comment's statement about its own
authorship at face value.

I also kept the first step, "decide where the policy demands disclosure", instead of
requiring a disclosure whenever any AI policy exists. `pkg-09` (fd) is a gold accept whose
policy asks for the tool to be stated "in the pull request" and says "no disclosure ask for
issue comments". A "disclose whenever there is a policy" version would wrongly reject it.

**Trade-offs**

The `ai-disclosure` revision loosens a check, so it could have let a `thread-convention`
reject through. I re-ran `--only pkg-03,pkg-20,pkg-04,pkg-09` before the confirming full run:

- `pkg-20` (ghostty, rejected only for the missing disclosure) and `pkg-04` (fzf, the other
  `thread-convention` package) were canaries for the 2-package category. Both stayed reject.
- `pkg-09` (fd, the PR-only disclosure accept) was a canary for the other direction. It
  stayed accept.
- The confirming full run kept `thread-convention 2/2`.

The cost I accept: because the check now takes a comment's statement about its own
authorship at face value, a comment that falsely claims "my own words" would pass this
check. The grader cannot verify authorship from the bundle text, and I would rather not
reject honest comments like `pkg-03`'s.

The confirming run also moved `pkg-14` from agree to disagree. That miss is on
`grounded-cause`, a check this revision did not touch: the row's wording is identical in the
Run 1 and Run 3 snapshots, and it passed the same package in Run 1. So I put the flip down
to grader variance on a borderline control, not to this trade-off. I left it as is because
the bar (19/20) and every category floor held. Rewording `grounded-cause` to say "a control
the plan explicitly explains does not contradict the cause" could fix `pkg-14`, but it
would risk letting `wrong-cause` plans through that explain away their contradicting
control (`pkg-16` and `calib-03` both confidently explain away evidence). That needs its own
canaried run before I adopt it.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.

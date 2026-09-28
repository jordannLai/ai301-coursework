# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

jordannLai

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/32#issuecomment-5818525392

````markdown
Hi! I'd like to pick this one up as my Unit 2 issue.

My read of the problem: `delete_profile` in `core/services/profile_service.py`
removes the profile's database rows, but nothing removes that profile's chunks
from the vector store. Since `ingestion/pipeline.py` writes a `profile_id` onto
every chunk it stores, the chunks for a deleted profile stay behind with their
`profile_id` intact, so retrieval can still surface content belonging to a
profile the API reports as deleted.

I haven't reproduced this yet — that's my next step. What I plan to do:

1. Set up the project locally following `docs/SETUP.md`.
2. Ingest a test profile and confirm its chunks are present in the vector store
   and returned by a retrieval query.
3. Call `DELETE /profiles/{profile_id}` for that profile
   (`api/routes/profiles.py`) and confirm the database rows are gone.
4. Query `rag/retriever/vector_store.py` again for chunks carrying that
   `profile_id`, and run the same retrieval query, to see whether the deleted
   profile's chunks are still stored and still retrievable.

I'll report back on this issue with what I actually observe — the environment,
the commands I ran, and the output — once I've run through those steps. If I
can't reproduce the behavior, I'll say so and describe what I saw instead.
````

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/32#issuecomment-5875475831

````markdown
Following up on my claim above — I reproduced this on current `main`. `DELETE /profiles/{profile_id}` returns `204` and clears every PostgreSQL row, but the profile's chunks stay in the vector store and remain retrievable by the deleted profile's ID.

**One caveat up front about how I set up the precondition:** no HTTP route populates the vector store, so I could not seed chunks through the API. `IngestionPipeline` is never instantiated anywhere in the application — `grep -rn "IngestionPipeline"` finds its definition in `ingestion/pipeline.py` and no call site in any application module — and `POST /profiles` just stores `resume_text` on the `Profile` row. I therefore called `IngestionPipeline.ingest_resume()` directly to create the chunks, wired the same way the app would wire it. Everything after that — the delete and both verifications — goes through the normal interfaces. Details and the full driver are below.

## Environment

| | |
|---|---|
| Commit | `2f4e82f52efbcfcc57d65b3fa5348672163ca088` |
| OS | macOS 26.5.2, arm64 |
| Python | 3.14.7 |
| PostgreSQL | `postgres:16-alpine`, server 16.15 (host port 5434 on my machine — 5433 was already taken locally) |
| Alembic | `002 (head)` |
| chromadb | 1.5.9, local `PersistentClient` at `.chromadb` |
| Embeddings | `LLM_PROVIDER=mock` → `MockEmbeddingProvider` (deterministic, no API key) |
| API | `uvicorn api.main:app` on `:8000` via `make run` |

Standard setup per `docs/SETUP.md` (`docker compose up -d`, `make setup`, `make run`). No application code was modified — `git diff --stat -- api core rag ingestion agent safety` is empty.

## What I checked in the code first

| Concern | Finding |
|---|---|
| Deletion logic | `core/services/profile_service.py:74-112` deletes `Review`, `IngestedSource` and `Profile` rows, then commits. No vector-store call anywhere in the function. |
| Route | `api/routes/profiles.py:195` — `DELETE /profiles/{profile_id}` → `204`, delegates to `delete_profile` and adds no vector-store step. |
| Vector store | `rag/retriever/vector_store.py:18` — `chromadb.PersistentClient(path=".chromadb")`, local on-disk. |
| Collection naming | `f"profile_{profile_id}"` (`rag/retriever/hybrid.py:54`). |
| `profile_id` on chunks | `ingestion/pipeline.py:98` (resume), `:175` (readme), `:251` (repo) put `profile_id` into chunk metadata; `ingestion/embeddings/batch_processor.py:113` passes the full metadata through to Chroma. |
| Available deletion primitive | Only `VectorStore.delete_by_source_id()` (`rag/retriever/vector_store.py:109`) — keyed by `source_id`, and nothing calls it from the deletion path. |

## Request formats

Worth noting since the three endpoints differ:

- `POST /auth/register` — `application/json`, body `{"email": ..., "password": ...}`
- `POST /auth/login` — **`application/x-www-form-urlencoded`**, fields `username` / `password` (JSON returns `422`)
- `POST /profiles` — **`multipart/form-data`**, fields `github_username`, `portfolio_url`, `resume_file`
- `DELETE /profiles/{profile_id}` — bearer token, no body

`@example.invalid` is rejected by the email validator (`value is not a valid email address: The part after the @-sign is a special-use or reserved name`), so the test account uses `example.com`.

## Reproduction driver

Run from the repo root with the venv active and the app running. Disposable data only: one throwaway user, one profile, one local Chroma collection.

<details>
<summary><code>repro32.py</code> — the script that produced the output below</summary>

```python
"""Controlled reproduction of issue #32.

DELETE /profiles/{profile_id} leaves the profile's embeddings in the vector store.

Disposable data only: a throwaway user, one profile, one local Chroma collection.
Does not modify any application code.
"""
import asyncio, json, secrets, sys, urllib.parse as up
import asyncpg, httpx

API = "http://localhost:8000"
MARKER = f"PATHREVIEW-REPRO-ISSUE32-{secrets.token_hex(4).upper()}"
EMAIL = f"repro-issue32-{secrets.token_hex(4)}@example.com"
PASSWORD = secrets.token_urlsafe(16)

def hr(t): print(f"\n{'='*70}\n{t}\n{'='*70}")

RESUME = f"""# Disposable Test Resume ({MARKER})

## Summary
{MARKER} synthetic candidate created only to reproduce issue 32.

## Experience
Built a {MARKER} widget pipeline in Python. Unique sentinel token: {MARKER}.

## Skills
Python, FastAPI, ChromaDB, {MARKER}
"""

def db_url():
    for line in open(".env"):
        if line.startswith("DATABASE_URL="):
            return line.strip().split("=", 1)[1]

async def pg():
    u = up.urlparse(db_url())
    return await asyncpg.connect(user=u.username, password=u.password,
                                 database=u.path.lstrip("/"), host=u.hostname, port=u.port)

async def main():
    print(f"marker      : {MARKER}")
    print(f"test user   : {EMAIL}")

    async with httpx.AsyncClient(timeout=30) as c:
        hr("STEP 1  Create disposable user + authenticate")
        r = await c.post(f"{API}/auth/register", json={"email": EMAIL, "password": PASSWORD})
        print(f"POST /auth/register -> {r.status_code}")
        r = await c.post(f"{API}/auth/login", data={"username": EMAIL, "password": PASSWORD})
        print(f"POST /auth/login    -> {r.status_code}")
        token = r.json()["access_token"]
        H = {"Authorization": f"Bearer {token}"}

        hr("STEP 2  Create profile via POST /profiles")
        r = await c.post(f"{API}/profiles", headers=H,
                         data={"github_username": f"repro-{MARKER.lower()}",
                               "portfolio_url": f"https://example.com/{MARKER}"},
                         files={"resume_file": ("disposable_resume.md", RESUME, "text/markdown")})
        print(f"POST /profiles      -> {r.status_code}")
        print(json.dumps(r.json(), indent=2)[:600])
        pid = r.json()["id"]
        print(f"\nPROFILE ID: {pid}")

    # ---- Ingest through the existing ingestion pipeline ----
    hr("STEP 3  Ingest resume via IngestionPipeline -> local .chromadb")
    from ingestion.embeddings.provider import MockEmbeddingProvider
    from ingestion.pipeline import IngestionPipeline
    from rag.retriever.vector_store import VectorStore

    COLL = f"profile_{pid}"          # naming convention from rag/retriever/hybrid.py:54
    vs = VectorStore(persist_dir=".chromadb")
    collection = vs.get_collection(COLL)
    provider = MockEmbeddingProvider()
    pipeline = IngestionPipeline(vector_db=collection, db_session=None, embedding_provider=provider)

    try:
        res = pipeline.ingest_resume(profile_id=pid, content=RESUME, filename="disposable_resume.md")
        print(f"ingest_resume -> source_id={res.source_id} chunk_count={res.chunk_count} skipped={res.skipped}")
        ingest_ok = True
    except Exception as e:
        print(f"ingest_resume FAILED: {type(e).__name__}: {e}")
        ingest_ok = False
    if not ingest_ok:
        sys.exit("BLOCKER: ingestion failed; see above")

    # ---- Pre-delete vector-store state ----
    hr("STEP 4  BEFORE delete: vector-store state")
    def vec_state(label):
        col = vs.get_collection(COLL)
        total = col.count()
        byprof = col.get(where={"profile_id": pid})
        print(f"[{label}] collection '{COLL}' exists, total chunks = {total}")
        print(f"[{label}] chunks with metadata profile_id == {pid}: {len(byprof['ids'])}")
        if byprof["ids"]:
            print(f"[{label}] sample chunk id  : {byprof['ids'][0]}")
            print(f"[{label}] sample metadata  : { {k:v for k,v in byprof['metadatas'][0].items() if k in ('profile_id','source_id','source_type','chunk_index')} }")
            print(f"[{label}] marker in text   : {MARKER in byprof['documents'][0]}")
        return total, len(byprof["ids"])

    before_total, before_prof = vec_state("BEFORE")

    def retrieval_check(label):
        from rag.retriever.hybrid import HybridRetriever
        from rag.retriever.keyword_search import KeywordSearcher
        qemb = provider.embed([MARKER])[0]
        direct = vs.query(qemb, COLL, n_results=5)
        print(f"[{label}] VectorStore.query  -> {len(direct)} hits"
              + (f", top score={direct[0]['score']:.4f}, marker in text={MARKER in direct[0]['text']}" if direct else ""))
        ks = KeywordSearcher()
        chunks = vs.get_collection(COLL).get(include=["documents", "metadatas"])
        if chunks["ids"]:
            ks.index([{"id": i, "text": d, "metadata": m}
                      for i, d, m in zip(chunks["ids"], chunks["documents"], chunks["metadatas"])])
        hr_ = HybridRetriever(vector_store=vs, keyword_searcher=ks)
        hyb = hr_.retrieve(query=MARKER, profile_id=pid, query_embedding=qemb, max_chunks=5)
        print(f"[{label}] HybridRetriever    -> {len(hyb)} hits"
              + (f", top score={hyb[0].get('score',0):.4f}" if hyb else ""))
        return len(direct), len(hyb)

    before_direct, before_hyb = retrieval_check("BEFORE")

    conn = await pg()
    prof_rows_before = await conn.fetchval("select count(*) from profiles where id=$1", __import__("uuid").UUID(pid))
    print(f"[BEFORE] postgres profiles rows for id: {prof_rows_before}")

    # ---- Delete ----
    hr("STEP 5  DELETE /profiles/{profile_id}")
    async with httpx.AsyncClient(timeout=30) as c:
        r = await c.delete(f"{API}/profiles/{pid}", headers=H)
        print(f"DELETE /profiles/{pid}")
        print(f"  -> HTTP {r.status_code}")
        print(f"  -> body: {r.text!r} (len {len(r.content)})")
        r2 = await c.get(f"{API}/profiles/{pid}", headers=H)
        print(f"GET /profiles/{pid} after delete -> HTTP {r2.status_code} {r2.text[:120]}")

    # ---- Post-delete Postgres ----
    hr("STEP 6  AFTER delete: PostgreSQL state")
    prof_rows_after = await conn.fetchval("select count(*) from profiles where id=$1", __import__("uuid").UUID(pid))
    src_rows_after = await conn.fetchval("select count(*) from ingested_sources where profile_id=$1", __import__("uuid").UUID(pid))
    rev_rows_after = await conn.fetchval("select count(*) from reviews where profile_id=$1", __import__("uuid").UUID(pid))
    print(f"profiles          rows for id: {prof_rows_after}")
    print(f"ingested_sources  rows for id: {src_rows_after}")
    print(f"reviews           rows for id: {rev_rows_after}")
    await conn.close()

    # ---- Post-delete vector store ----
    hr("STEP 7  AFTER delete: vector-store state (same profile_id, same store)")
    vs_after = VectorStore(persist_dir=".chromadb")
    names = [c.name if hasattr(c, "name") else c for c in vs_after.client.list_collections()]
    print(f"collections present: {names}")
    after_total, after_prof = vec_state("AFTER")
    after_direct, after_hyb = retrieval_check("AFTER")

    hr("RESULT")
    print(f"profile_id                      : {pid}")
    print(f"postgres profile rows  before/after : {prof_rows_before} / {prof_rows_after}")
    print(f"chroma chunks (profile_id) before/after : {before_prof} / {after_prof}")
    print(f"vector query hits      before/after : {before_direct} / {after_direct}")
    print(f"hybrid retrieve hits   before/after : {before_hyb} / {after_hyb}")
    reproduced = prof_rows_after == 0 and after_prof > 0 and after_direct > 0
    print(f"\nISSUE #32 REPRODUCED: {reproduced}")

asyncio.run(main())
```

</details>

`.chromadb/` did not exist before the run, so the store held nothing else. The profile in this run was `9e682e8a-dc8b-4709-af66-5d88e7edc17d`, sentinel `PATHREVIEW-REPRO-ISSUE32-EBBDB82E`.

## Output

**Steps 1–2 — create the user and profile**

```
POST /auth/register -> 200
POST /auth/login    -> 200
POST /profiles      -> 200
{
  "id": "9e682e8a-dc8b-4709-af66-5d88e7edc17d",
  "user_id": "f90904fb-6417-4573-ba66-cf264d2437e4",
  "github_username": "repro-pathreview-repro-issue32-ebbdb82e",
  "portfolio_url": "https://example.com/PATHREVIEW-REPRO-ISSUE32-EBBDB82E",
  "created_at": "2026-09-24T22:08:14.356491Z",
  "resume_filename": "disposable_resume.md"
}
```

**Step 3 — ingest**

```
[info] created_collection  collection_name=profile_9e682e8a-dc8b-4709-af66-5d88e7edc17d
[info] Starting resume ingestion  profile_id=9e682e8a-... source_id=resume_9e682e8a-..._c29a4073aaeb68f9
[warning] Could not check if source already ingested  error="'NoneType' object has no attribute 'query'"
[info] Resume parsed successfully  sections=['Summary', 'Skills', 'Experience']
[info] Resume chunked successfully chunk_count=1
[info] Batch embedding processing complete stored_count=1
ingest_resume -> source_id=resume_9e682e8a-dc8b-4709-af66-5d88e7edc17d_c29a4073aaeb68f9 chunk_count=1 skipped=False
```

The warning is `_check_skip` reacting to `db_session=None`; the pipeline catches it and proceeds, which is why passing `None` is harmless here.

**Step 4 — BEFORE delete**

```
[BEFORE] collection 'profile_9e682e8a-dc8b-4709-af66-5d88e7edc17d' exists, total chunks = 1
[BEFORE] chunks with metadata profile_id == 9e682e8a-dc8b-4709-af66-5d88e7edc17d: 1
[BEFORE] sample chunk id  : resume_9e682e8a-dc8b-4709-af66-5d88e7edc17d_c29a4073aaeb68f9_chunk_0
[BEFORE] sample metadata  : {'chunk_index': 0, 'profile_id': '9e682e8a-dc8b-4709-af66-5d88e7edc17d',
                             'source_type': 'resume', 'source_id': 'resume_9e682e8a-..._c29a4073aaeb68f9'}
[BEFORE] marker in text   : True
[BEFORE] VectorStore.query  -> 1 hits, top score=0.4916, marker in text=True
[BEFORE] HybridRetriever    -> 1 hits, top score=0.7000
[BEFORE] postgres profiles rows for id: 1
```

**Step 5 — the delete**

```
DELETE /profiles/9e682e8a-dc8b-4709-af66-5d88e7edc17d
  -> HTTP 204
  -> body: '' (len 0)
GET /profiles/9e682e8a-dc8b-4709-af66-5d88e7edc17d after delete -> HTTP 404 {"detail":"Profile not found"}
```

**Step 6 — PostgreSQL after the delete**

```
profiles          rows for id: 0
ingested_sources  rows for id: 0
reviews           rows for id: 0
```

**Step 7 — vector store after the delete, same store and same profile ID**

```
collections present: ['profile_9e682e8a-dc8b-4709-af66-5d88e7edc17d']
[AFTER] collection 'profile_9e682e8a-dc8b-4709-af66-5d88e7edc17d' exists, total chunks = 1
[AFTER] chunks with metadata profile_id == 9e682e8a-dc8b-4709-af66-5d88e7edc17d: 1
[AFTER] sample chunk id  : resume_9e682e8a-dc8b-4709-af66-5d88e7edc17d_c29a4073aaeb68f9_chunk_0
[AFTER] marker in text   : True
[AFTER] VectorStore.query  -> 1 hits, top score=0.4916, marker in text=True
[AFTER] HybridRetriever    -> 1 hits, top score=0.7000
```

To rule out in-process caching, I re-opened the store in a **fresh Python process** and queried with wording that does not contain the sentinel:

```python
import sys
pid, marker = sys.argv[1], sys.argv[2]
from rag.retriever.vector_store import VectorStore
from ingestion.embeddings.provider import MockEmbeddingProvider
vs = VectorStore(persist_dir=".chromadb")
print("collections on disk:", [c.name if hasattr(c,'name') else c for c in vs.client.list_collections()])
col = vs.get_collection(f"profile_{pid}")
got = col.get(where={"profile_id": pid})
print(f"orphaned chunks for deleted profile: {len(got['ids'])}")
for i, d in zip(got['ids'], got['documents']):
    print(f"  id={i}")
    print(f"  marker present: {marker in d}")
    print(f"  text excerpt : {d[:150].strip()!r}")
res = vs.query(MockEmbeddingProvider().embed(["widget pipeline built in Python"])[0], f"profile_{pid}", n_results=3)
print(f"semantic query (non-marker wording) -> {len(res)} hits; leaked text: {res[0]['text'][:90].strip()!r}" if res else "no hits")
```

```
collections on disk: ['profile_9e682e8a-dc8b-4709-af66-5d88e7edc17d']
orphaned chunks for deleted profile: 1
  id=resume_9e682e8a-dc8b-4709-af66-5d88e7edc17d_c29a4073aaeb68f9_chunk_0
  marker present: True
  text excerpt : 'Disposable Test Resume (PATHREVIEW-REPRO-ISSUE32-EBBDB82E) Summary ...'
semantic query (non-marker wording) -> 1 hits;
  leaked text: 'Disposable Test Resume (PATHREVIEW-REPRO-ISSUE32-EBBDB82E) Summary PATHREVIEW-REPRO-ISSUE3'
```

## Before and after

| Measurement | Before delete | After delete |
|---|---|---|
| Postgres `profiles` rows | 1 | **0** |
| Postgres `ingested_sources` / `reviews` rows | 0 / 0 | 0 / 0 |
| Chroma chunks with `profile_id` | 1 | **1** |
| `VectorStore.query` hits | 1 | **1** |
| `HybridRetriever.retrieve` hits | 1 | **1** |
| Resume text recoverable from store | yes | **yes** |

## Expected vs observed

**Expected:** after `DELETE /profiles/{profile_id}` returns `204`, the profile's chunks are removed from the vector store along with its database rows — a retrieval scoped to that profile returns nothing, and the resume text is no longer recoverable.

**Observed:** the delete returns `204 No Content` and removes every PostgreSQL row, but the vector store is untouched. The collection `profile_<id>`, its chunk, the `profile_id` metadata and the full resume text all survive on disk, across a fresh process, and stay retrievable through both `VectorStore.query()` and `HybridRetriever.retrieve()` for the deleted profile's ID.

## Where it comes from

`delete_profile` (`core/services/profile_service.py:74-112`) performs only ORM deletes — `Review`, `IngestedSource`, `Profile` — then commits. It never imports or calls `VectorStore`, and `api/routes/profiles.py:195` adds no vector-store step, so nothing in the deletion path touches `.chromadb`.

The data needed is already there: every chunk carries `profile_id` in its metadata (`ingestion/pipeline.py:98`) and chunks are partitioned per profile as `profile_{profile_id}` (`rag/retriever/hybrid.py:54`). What's missing is a profile-scoped deletion primitive — `VectorStore` currently exposes only `delete_by_source_id` (`rag/retriever/vector_store.py:109`).

Practical impact: a deleted user's resume text stays indexed and semantically retrievable indefinitely, which reads as a data-retention problem rather than a stale-cache one.

## Scope of this reproduction

Confirming what this does and does not show. The chunks were created by calling `IngestionPipeline.ingest_resume()` directly, because no HTTP route populates the vector store — so this reproduces the deletion gap, not an end-to-end user journey. Everything downstream of that seeding used the normal interfaces: the `DELETE` went through the API, and both verifications used the app's own `VectorStore` and `HybridRetriever`. Whether chunks arrive via the pipeline or any future route, `delete_profile` has no code path that removes them.

All test data was disposable and no application code was modified.
````

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

Three runs, in order:

1. **First complete run — `agreement: 19/20`.** Category line showed `disclosure 0/1`: my
   `Repository conventions` check passed `pkg-20` (gold `reject`), whose repo facts state
   "All AI usage in any form must be disclosed, stating the tool used and the extent of the
   assistance". My check had no rule for a repo that *requires* disclosure, so a package with
   no disclosure text drew no penalty. The 19th–20th points came from `clear-accept 8/8`.
2. **Targeted run — `--only pkg-01,pkg-20`, agreement 2/2.** After rewriting
   `Repository conventions` around the three-way (A)/(B)/(C) policy classification, I re-ran
   the fixed package plus `pkg-01` as a canary to confirm the new disclosure rule had not
   started rejecting clean accepts. `pkg-20` flipped to `reject`, `pkg-01` stayed `accept`.
3. **Final complete run — `agreement: 19/20 scored items  (bar: 18/20: PASS)`**, with every
   category satisfied: `clear-accept 7/8  disclosure 1/1  no-evidence 4/4
   unfollowable-comms 3/3  wrong-target 4/4`. The remaining miss moved from `pkg-20` to
   `pkg-10`.

The last score, 19/20, matches the agreement line in the committed `eval-run.txt`.

**Package analysis**

**`pkg-10`** (`starship/starship#7648`, category `clear-accept`). Gold label: **accept**.
My rubric: **reject**, recorded in `eval-run.txt` as
`pkg-10  accept  reject   NO     failed: Issue-specific evidence`.

The package is an honest cannot-reproduce. The contributor ran the report's exact symlink
layout and config on Linux + zsh, showed the resulting prompt
(`monorepo/packages/app-dir on  master`), stated that `starship explain` still lists the
`directory` module, and named both the environment gap (macOS + fish 4.7.1 versus
Ubuntu 24.04 + zsh 5.9) and a hypothesis for why the shell matters — that fish resolves
`PWD` logically, which is what would drive `contract_repo_path` to return `None`.

My `Issue-specific evidence` check rejected it on this clause:

> Fail if the artifacts establish only an adjacent problem, an unrelated setup failure, or a
> different behavior.

The artifacts here show the module *rendering correctly* — literally "a different behavior"
than the vanishing module the issue describes — so the clause fires on exactly the evidence
that makes an honest cannot-reproduce trustworthy. My `Honest outcome` check carves out that
case ("A cannot-reproduce result passes when the report documents a genuine attempt against
the correct target and provides evidence of what happened instead"), and
`references/evidence-guide.md` agrees ("For an honest cannot-reproduce result, the artifacts
should show that the correct behavior was tested and what occurred instead"). The carve-out
never reaches the verdict, because both checks are `required` and `Issue-specific evidence`
fails first. The gold label is right and my rubric is wrong: the fix is to scope that fail
clause to reports *claiming* reproduction, rather than to widen `Honest outcome`.

**Check rationale**

Quoted exactly as it reads now in `tools/repro-check/rubric.md`:

> | Repository conventions | Repo-facts block: the contribution and AI-use policy, classified as (A) an explicit disclosure requirement, (B) a conditional AI policy, or (C) no AI policy. Claim comment and repro report: the AI-use disclosure text, claims about completed work, and promises. Voice guide: required communication conventions. | Pass if the comments comply with applicable contribution rules and do not falsely claim completed work or promise an unverified fix. Grade AI use by which policy the repo facts state. (A) Explicit disclosure requirement (the policy requires AI usage to be disclosed, for example "all AI usage in any form must be disclosed, stating the tool used and the extent of the assistance"): pass only if the claim comment or the repro report actually carries a disclosure in the form the policy asks for. Treat a submitted package as AI-assisted work, so an absent disclosure is a fail, not evidence that no AI was used. (B) Conditional AI policy (AI permitted subject to conditions such as testing, human review, or limits on generated code): pass if the comments show the stated conditions were met; an absent disclosure fails only when the policy makes disclosure one of those conditions. (C) No AI policy in the repo facts: silence about AI use is not a failure and does not create a disclosure requirement. | required |

It reads that way because the first complete run scored `disclosure 0/1`. The earlier version
asked only whether the comments "follow the repository's contribution rules", which left the
model to decide for itself whether silence about AI counted as a violation — and on `pkg-20`
it decided silence was fine, passing a package the gold label rejects.

What I rejected first: a blanket rule requiring an AI disclosure in every package. That would
have bought `pkg-20` at the cost of the fourteen packages whose repo facts state no AI policy,
turning a single category miss into a much larger one.

What I adopted instead classifies the *policy* before grading the *comment*, so the burden of
proof follows the repo. The clause that actually does the work on `pkg-20` is
"Treat a submitted package as AI-assisted work, so an absent disclosure is a fail, not
evidence that no AI was used" — without it, an empty comment is indistinguishable from an
honest one. Case (C) is what protects the no-policy majority: "silence about AI use is not a
failure and does not create a disclosure requirement".

**Trade-offs**

**A package whose result it changes, plus the canary that bounded the change.** The
`Repository conventions` rewrite flipped `pkg-20` from `accept` to `reject` (correct). Because
case (A) instructs the grader to treat an absent disclosure as a fail, I re-ran `pkg-01` with
`--only pkg-01,pkg-20` to check the new rule had not leaked into repos with no AI policy.
Agreement was 2/2: `pkg-01` still `accept`. Across the final full run the no-policy packages
held, `disclosure` went 0/1 → 1/1, and no other category moved.

**A case I accept it will miss.** The cost landed on a different check. My
`Issue-specific evidence` pass condition reads:

> Pass if the evidence directly tests the behavior described in the issue and supports the reported result. Fail if the artifacts establish only an adjacent problem, an unrelated setup failure, or a different behavior.

Scoring "a different behavior" as a failure is what catches the four `wrong-target` packages
(`pkg-02`, `pkg-08`, `pkg-16`, `pkg-17`, all `reject`, all correct), and it is the same clause
that sinks `pkg-10`. As written I cannot separate "the artifacts show an adjacent bug" from
"the artifacts show the target behavior working, which is the whole point of a
cannot-reproduce". I am keeping the strict wording for now: 4 correct rejects against 1
incorrect reject is the better trade at this bar, and the honest cannot-reproduce case is
rarer in the set than the wrong-target case. The narrower fix — limiting the clause to reports
that claim successful reproduction — is what I would change next, and it is why the final run
is 19/20 rather than 20/20.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.

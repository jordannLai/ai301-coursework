# Plan: issue #32 — `DELETE /profiles/{profile_id}` leaves the profile's embeddings in the vector store

- **Issue:** https://github.com/codepath/pathreview-ai301-fa26-s3/issues/32
- **Reproduction:** my repro comment on #32 (commit `2f4e82f`, chromadb 1.5.9, local `PersistentClient` at `.chromadb`, `LLM_PROVIDER=mock`)
- **Branch:** `fix/32-vector-store-cleanup` (on my fork)

## Diagnosis

`delete_profile` (`core/services/profile_service.py:74-112`) only performs ORM deletes (`Review`, `IngestedSource`, `Profile`) and commits. It never calls the vector store, and `DELETE /profiles/{profile_id}` (`api/routes/profiles.py:195`) adds no vector-store step. So nothing in the deletion path touches the Chroma collection that holds the profile's chunks.

My reproduction shows exactly this:

| Measurement | Before delete | After delete |
|---|---|---|
| Postgres `profiles` rows | 1 | **0** |
| Chroma chunks with `profile_id` | 1 | **1** |
| `VectorStore.query` hits | 1 | **1** |
| `HybridRetriever.retrieve` hits | 1 | **1** |

The chunk survived in a fresh Python process and was still returned by a semantic query that didn't contain the sentinel token, so this isn't an in-process cache effect. The store-side data needed for the fix already exists. Chunks are partitioned per profile into a collection named `profile_{profile_id}` (`rag/retriever/hybrid.py:54`, the only collection retrieval reads for a profile), and every chunk also carries `profile_id` in its metadata (`ingestion/pipeline.py:98`). What's missing is (a) a profile-scoped deletion primitive on `VectorStore`, which currently has only `delete_by_source_id` (`rag/retriever/vector_store.py:109`), and (b) a call to it from `delete_profile`.

## Scope

**In scope:** one bounded change. After deleting a profile, its vector-store chunks are deleted too.

- A `VectorStore` method that removes a profile's `profile_{profile_id}` collection and is a no-op when that collection doesn't exist.
- Calling it from `delete_profile` as part of the same deletion, with a failure rolling the DB delete back.
- Unit tests for both, plus a re-run of my reproduction.

**Not in scope:**

- Wiring `IngestionPipeline` into an HTTP route. No route populates the vector store today. That's a separate gap my repro worked around, not this bug.
- Switching `VectorStore` from the local `PersistentClient(".chromadb")` to the `vector_db_url` HTTP server in `core/config.py`. It's a real mismatch, but it's a separate configuration issue.
- Refactoring the `f"profile_{profile_id}"` naming in `hybrid.py` into a shared helper, and any change to `HybridRetriever` or `delete_by_source_id`.
- Converting the ORM deletes to DB-level `ON DELETE CASCADE`, or changing the route's status codes.
- Any `# noqa` or seeded-defect lines (e.g. `hybrid.py`'s issue #6 marker).

## Files to change

1. `rag/retriever/vector_store.py`: add `VectorStore.delete_profile_chunks(profile_id) -> int`.
2. `core/services/profile_service.py`: `delete_profile` accepts an optional `vector_store` and calls the new method before `db.commit()`.
3. `tests/unit/test_vector_store.py` (new): tests for the new method against a temporary Chroma directory.
4. `tests/unit/test_profile_service.py` (new): tests for `delete_profile`'s ordering and rollback, with the DB session and vector store mocked (same `AsyncMock` pattern as `tests/unit/test_review_service.py`).

`api/routes/profiles.py` needs no logic change. The route already delegates to `delete_profile`, maps exceptions to 500, and returns 204. At most its docstring gains "and the profile's vector-store chunks".

## Implementation approach

1. **`VectorStore.delete_profile_chunks(profile_id: str) -> int`** in `rag/retriever/vector_store.py`:
   - Build the name `f"profile_{profile_id}"`, the same convention `HybridRetriever` reads.
   - Look the collection up with `self.client.get_collection(name)`, **not** `self.get_collection`, which would create an empty one. If Chroma raises `NotFoundError`, log `profile_chunks_absent` and return `0`. I checked that chromadb 1.5.9 raises `NotFoundError: Collection [...] does not exist` here. This is the common case today, because no route ingests anything.
   - Otherwise record `collection.count()`, call `self.client.delete_collection(name)`, log `deleted_profile_chunks` with the count, and return it.
   - Google-style docstring, per `docs/CONTRIBUTING.md`.
2. **`delete_profile`** in `core/services/profile_service.py`:
   - New keyword parameter `vector_store: VectorStore | None = None`; when `None`, use `VectorStore()`, which has the same default `.chromadb` path as the retriever and the pipeline. Existing callers (the route) stay unchanged.
   - Inside the existing `try`, after the ORM deletes and **before** `await db.commit()`, call `vector_store.delete_profile_chunks(str(profile_id))`. If it raises, the existing `except` logs `profile_cascade_delete_failed`, rolls back, and re-raises. The route turns that into a 500, the profile stays intact, and the user can retry. Cleaning up before the commit means a failure can never leave the reported state (DB rows gone, chunks orphaned).
   - Not-found and not-owned profiles return `False` before any vector-store call, as today.
3. Add the tests below, then run `make check` and `make test-unit`.
4. Commit as `fix(api): delete a profile's vector-store chunks on profile deletion` with `Fixes #32` (Conventional Commits, per `docs/CONTRIBUTING.md`).

## Test plan

What I'll observe flip from broken to fixed is the same measurement as my reproduction: **chunks for the deleted profile, before → after**.

1. **Re-run my repro driver (`repro32.py`)** against the fixed branch, with the same setup and a fresh `.chromadb`.
   - Today: chunks after delete = 1, `VectorStore.query` hits after = 1, `HybridRetriever` hits after = 1, and `ISSUE #32 REPRODUCED: True`.
   - Expected after the fix: `DELETE` still returns `204` and Postgres rows still go to 0. `collections present` no longer lists `profile_<id>`. Chunks after delete = **0**, both retrieval hit counts = **0**, and the driver prints `ISSUE #32 REPRODUCED: False`.
   - The fresh-process check should print `orphaned chunks for deleted profile: 0` and `no hits`.
   - Caveat: the driver's post-delete checks call `VectorStore.get_collection`, which recreates an empty `profile_<id>` collection. The `collections present` line prints before that recreate, so it's the meaningful listing. The later chunk and hit counts read 0 from the empty, recreated collection. For the PR evidence I'll switch the driver's after-delete probe to `vs.client.get_collection`, so a missing collection shows up as `NotFoundError` instead of being recreated.
2. **`tests/unit/test_vector_store.py`** (`@pytest.mark.unit`, `tmp_path` for the Chroma directory):
   - `test_delete_profile_chunks_removes_profile_collection`: add 2 chunks to `profile_<id>` with `profile_id` metadata. Call the method. Assert it returns `2`, that `profile_<id>` is no longer in `client.list_collections()`, and that a `profile_id` query finds nothing.
   - `test_delete_profile_chunks_leaves_other_profiles`: seed profiles A and B, delete A, assert B's chunk count is unchanged.
   - `test_delete_profile_chunks_missing_collection_is_noop`: on an empty store, assert it returns `0` without raising and doesn't create the collection.
3. **`tests/unit/test_profile_service.py`** (`@pytest.mark.unit`, `@pytest.mark.asyncio`, mocked `db` and `vector_store`):
   - Calls `delete_profile_chunks(str(profile_id))` exactly once and before `db.commit`.
   - When `delete_profile_chunks` raises: `db.rollback` is awaited, `db.commit` is never awaited, and the exception propagates.
   - When the profile isn't found or isn't owned: returns `False` and the vector store is never called.
4. `make check` (ruff, black, mypy) and `make test-unit` are green. All five CI jobs are green on the PR. There's no existing `xfail` marker for #32 (I grepped `tests/` for one), so there's no marker to remove.

## Risks / unknowns

- **Is the `profile_{profile_id}` collection the only place a profile's chunks live?** Retrieval reads only that collection, and my repro stored there. But `IngestionPipeline` takes whatever collection its caller passes, and no app code constructs one yet. If a future wiring writes to a shared collection, dropping the per-profile collection would miss those chunks. I'll call this out in the PR. If review prefers it, the method can additionally `collection.delete(where={"profile_id": ...})` on a named shared collection, but I won't add that speculatively.
- **Two stores in config.** `VectorStore` uses local `.chromadb`, while `core/config.py` has `vector_db_url` for a Chroma server that nothing uses. My fix deletes from the store the app actually reads and writes today (`.chromadb`). If the app later moves to the server, the cleanup moves with `VectorStore`. I'm flagging this, not fixing it.
- **New import edge.** `core/services` doesn't import from `rag/` anywhere today. Injecting `VectorStore` into `delete_profile` adds that dependency. If maintainers would rather keep `core` free of `rag`, the same call can move into the route handler (`api/routes/profiles.py`) before the service commits. I'll ask in the PR rather than guess.
- **Not atomic across two stores.** If the Chroma delete succeeds and then `db.commit()` fails, the profile row survives without its chunks. I chose that over the reverse because chunks are derived data that can be re-ingested, while orphaned chunks are the data-retention problem this issue reports. I haven't measured anything here. It's a design trade-off for review.
- **Untested environments.** I've only run this on macOS arm64 with chromadb 1.5.9 and the mock embedding provider. I haven't tested against the dockerized Chroma server or a real embedding provider.

## Deviations

The fix itself went in as planned. I touched the same four files with the same approach: `VectorStore.delete_profile_chunks` drops `profile_{profile_id}` and returns 0 on `NotFoundError`, and `delete_profile` calls it before `db.commit()` through an optional `vector_store` argument, so a failure still rolls back. I left `api/routes/profiles.py` untouched, docstring included, because the route needed no change. All six planned unit tests are in place. They pass on the fix and all six fail on the original code.

Three small differences from the test plan, none of which change the fix:

1. **Repro probe.** The plan said I'd switch the driver's after-delete probe to `vs.client.get_collection`. Instead I re-ran `repro32.py` unchanged, so the numbers compare directly with my Unit 2 run. I added a separate fresh-process probe that uses `client.get_collection` (which never creates a collection) and counts chunks tagged with the profile ID across every collection.
   - Original code, same day: chunks 1 → 1, hits 1 → 1, `ISSUE #32 REPRODUCED: True`.
   - Fixed code: `collections present: []` right after the delete, chunks 1 → 0, `VectorStore.query` and `HybridRetriever` hits 1 → 0, `REPRODUCED: False`. The probe finds 0 tagged chunks and 0 semantic hits.
   - The probe still lists a `profile_<id>` collection. That's the empty one the driver's later `get_collection` call recreates, which I flagged in the caveat above. It holds 0 chunks.
2. **`make typecheck` locally.** It fails before checking any project code, because my venv is Python 3.14 while mypy is pinned to `python_version = "3.11"`, and numpy's 3.14 stubs use 3.12+ syntax. With `--python-version 3.14`, mypy reports no errors in the changed files. The 3 errors it does report are in test files I didn't touch. CI runs 3.11, so I'll rely on the CI `typecheck` job for the official result.
3. **Lint and format.** ruff flagged one SIM117 (nested `with`) in my new test, and black reformatted the two new test files. I fixed both by hand and didn't run `--fix` or `black .` on the repo. `make lint` and `black --check .` now pass, and `make test-unit` passes with 381 passed and 53 xfailed (375 before, plus my 6).

None of these change the diagnosis, the scope, the files, or the approach, so I didn't treat them as material.

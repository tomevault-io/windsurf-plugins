---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies (requires Python 3.14)
uv sync

# Run the app (must be run from the directory containing spruceup_pipeline.py)
uv run spruceup start

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_sync_engine.py

# Run a specific test
uv run pytest tests/test_sync_engine.py::test_reconcile_new_file
```

Required env vars for the pipeline: `PG_CONNSTR`, and an embedder API key (e.g. `OPENAI_API_KEY`). Copy credentials into `.env`; `spruceup_pipeline.py` calls `dotenv.load_dotenv()` at import time.

## Architecture

SpruceUp is a document ingestion daemon. It watches source connectors for file changes, transforms documents into chunks, embeds them, and keeps a target vector store in sync.

### Pipeline file (`spruceup_pipeline.py`)

The user-authored entry point. The CLI (`spruceup start`) imports it dynamically from the CWD. It must define a `config` variable returned by `define_config()`:

```python
config = define_config(
    sources=[LocalFilesSource(watched_dir="example/data_corpus")],
    target=PgVectorTarget(connstr=..., table="data_chunks", schema=LectureChunk, vector_column="chunk_embedding"),
    embedder=OpenAIEmbedder(api_key=..., model="text-embedding-3-small"),
    transform=build_lecture_chunks,  # async fn(*, file_props: FileProps, embed) -> list[schema]
    cache_files=False,  # optional; True caches raw file content in the manifest (default False)
)
```

`define_config()` validates types eagerly at import time. `validate_pipeline()` in `cli.py` checks the contract exists before starting the event loop.

### Runtime flow (`app.py`)

On startup, `app.run(pipeline)` compares persisted fingerprints in the Manifest against the current config. Any mismatch triggers a **full reindex** (all files re-fetched, re-transformed, re-upserted) instead of incremental sync:
1. Transform function body changed (source hash)
2. Any `@memoize`-decorated function changed
3. Embedding model changed
4. Embedding dimensions changed
5. Target identity changed — `target.identity()`, a credential-free string (host/db/table or index/collection)
6. Schema changed — `hash_schema()` over field names+types and the designated `vector_column`

Signals 3–4 additionally **flush the embedding cache** (`embeddings_invalidated`). Signals 4–6 are **structural** and additionally **drop + recreate** the target table/index before reingest (`ensure_table_exists(recreate=True)`) — chosen over in-place migration because reingest must re-embed everything anyway.

On any mismatch, every file row is marked `needs_reindex` and the new fingerprints are persisted immediately (mark first, persist second — a crash in between just re-marks on the next start). A file stays `needs_reindex` until a sync **succeeds**; failures and restarts don't clear it. `SyncEngine.reconcile` pushes **all** chunks of a `needs_reindex` file instead of the diff (config changes don't alter chunk hashes, so the diff can't see them). `needs_reindex` files are re-enqueued at every startup and retried by the sync sweeper, so an interrupted reindex resumes where it left off.

Then it launches three concurrent asyncio tasks:

| Task | Role |
|------|------|
| `Monitor` | Runs all watchers; each watcher does a catch-up scan then enters a watch loop |
| `Coordinator` | Dequeues `SyncTask` objects and processes them (up to 32 concurrent) |
| `SyncSweeper` | Retries `failed` and `needs_reindex` files every 60 seconds |

### File change lifecycle

```
Source watcher → DebounceQueue → Coordinator
                                     ↓
                              source.fetch() → SpruceFile
                                     ↓
                              transform(file_props, embed) → list[UserChunk]
                                     ↓
                              SyncEngine.reconcile() → chunk diff → target.sync()
                                     ↓
                              Manifest.set_sync_state("synced")
```

`DebounceQueue` (wraps `asyncio.Queue`) evicts any already-queued task for the same `file_id` when a newer task arrives, preventing redundant processing. **Tradeoff:** to evict the superseded task it reaches into `asyncio.Queue` internals (`_queue`, `_unfinished_tasks`), so it's coupled to the CPython queue implementation and could break on a stdlib change.

### Manifest (`manifest.py`)

A local SQLite database (`spruceup_manifest.db`) that is the source of truth for:
- Registered data sources and their state (e.g. Google Drive page tokens)
- File rows: content hash, raw content (only when `cache_files=True`), sync state (`needs_reindex` / `in_flight` / `synced` / `failed`)
- Chunk rows: `(file_id, user_chunk_object_hash)` pairs for diffing
- Memoize cache: `(file_id, fn_hash, args_hash) → result`
- Embedding cache: `(file_id, chunk_text_hash) → embedding bytes`
- Config state: `embedding_model`, `embedding_dimensions`, `target_identity`, `schema_fingerprint`

Opened with `autocommit=True`; use `manifest.transaction()` only when multiple writes must be atomic.

### Connector ABCs (`connectors/base.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpruceUp-ai/SpruceUp](https://github.com/SpruceUp-ai/SpruceUp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->

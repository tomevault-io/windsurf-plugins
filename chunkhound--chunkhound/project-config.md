---
trigger: always_on
description: This file documents the design decisions and known gaps behind the Rust indexing
---

# Rust indexing pipeline (`chunkhound_native`)

This file documents the design decisions and known gaps behind the Rust indexing
crate rooted here. It exists so an agent (or human) touching this code doesn't
have to re-derive *why* things are shaped this way from scattered comments and
git history. Read this before changing anything under `src/`, and treat items 4-5
as relevant even when editing the Python-side files they name.

**Maintenance policy:** update this file when the orchestration/threading model
changes, a new DB backend gains Rust support, a fallback condition changes, or a
"deliberate exclusion" below gets resolved (move it out of the gaps table). Do
NOT log individual bug fixes here — those belong in commit messages and
`CHANGELOG.md`. This doc records *why*, not a changelog. If you notice an entry
below is stale, fix it rather than leaving it.

## 1. Scope: orchestration with native embedding adapters

The Rust pipeline does NOT parse code. It owns file scanning, diffing,
threading/scheduling, DuckDB writes, and native embedding requests for the
OpenAI/Azure OpenAI and VoyageAI providers. All other provider names (or an
unrecognized provider/configuration combination) fall back to the existing
Python embedding callback:

- `IndexingPipeline.run()` takes `parse_batch_callback` and `embed_batch_callback`
  as Python callables and invokes the parse callback from a dedicated Rust
  thread. The embedding callback remains available as the per-run fallback.
- Native adapters live under `embed/`; the Python side of the fallback lives in
  `chunkhound/pipeline_bridge.py` (`embed_batch_callback`).
- Parsing remains entirely in the existing Python implementation: language
  detection and tree-sitter parsing happen in `parse_file_callback`.

This is a deliberate architecture, not a partial parsing migration in
progress. Embedding deduplication/reuse is explicitly out of scope here and
does not currently exist anywhere in the pipeline.

## 2. Module map

| Path | Purpose |
|---|---|
| `lib.rs` | `#[pymodule]` entry point; `scan_files()` — parallel file discovery via the `ignore` crate |
| `error.rs` | `DbError`/`ScanError` → `PyErr` conversions |
| `embed/{mod,callback,common,factory,openai,voyageai,retry,token}.rs` | Embedding trait, Python fallback, native providers, retries, and token-aware batching. `common.rs` holds the shared client-pool/validate/retry/sanitize scaffolding both native providers delegate to |
| `types.rs` | DB-facing serde structs shared across the PyO3 boundary |
| `db/mod.rs` | `DbBackend` trait, `DbConfig`, `create_backend()` |
| `db/duckdb_backend/mod.rs` | `DuckDbHnswBackend` struct, open/close lifecycle |
| `db/duckdb_backend/schema.rs` | DDL for files/chunks/embeddings tables |
| `db/duckdb_backend/write.rs` | Batched upserts/inserts, single-transaction batch writes |
| `db/duckdb_backend/read.rs` | `read_file_states()` (diff-phase snapshot), disk usage check |
| `db/duckdb_backend/hnsw.rs` | HNSW vector index drop/discover/rebuild (VSS extension) |
| `db/duckdb_backend/compaction.rs` | ATTACH+INSERT-SELECT DB compaction |
| `db/duckdb_backend/recovery.rs` | 3-phase swap-intent crash recovery for compaction |
| `pipeline/pipeline.rs` | `IndexingPipeline` `#[pyclass]` — orchestrates diff → parse → embed → store |
| `pipeline/differ.rs` | `compute_diff()` — filesystem vs. DB-snapshot diffing (mtime/hash based) |
| `pipeline/config.rs` | `PipelineConfig::from_py_dict()` |
| `pipeline/types.rs` | Internal `ParsedFile`/`NewChunk` — never exposed to Python |
| `pipeline/report.rs` | `PipelineReport` `#[pyclass]` returned to Python |
| `pipeline/parse_call_config.rs` | `ParseCallConfig` `#[pyclass]` passed into the Python parse callback |
| `analytics/{mod,recorder,command,identity,repository,s3}.rs` | Per-user usage analytics: `AnalyticsRecorder` `#[pyclass]` (local JSONL buffer, background flush thread, SigV4 S3 upload); `command.rs`'s handle-based table (no contextvars — callers pass an explicit `u64` handle) is what lets both ordinary Python call sites and the native embed adapters record into the same rollup without a cross-thread propagation problem |

## 3. PyO3 boundary design decisions

Baseline rules (`#![forbid(unsafe_code)]`, no `.unwrap()` at the boundary, no
borrowing `&str` across `py.allow_threads()`, always `allow_threads` for
CPU/IO-bound work) are defined once in the root `AGENTS.md` under `RUST_RULES`
— follow those, don't re-derive them here.

**Threading model** (`pipeline/pipeline.rs:416-440`): three persistent OS
threads — parse, embed, store — connected by two bounded `mpsc` channels
(capacity 2 each). While the store thread writes batch N to DuckDB (and, on the
final batch, rebuilds HNSW indexes and compacts), the embed thread is already
embedding batch N+1 and the parse thread is already parsing batch N+2. The
bounded channels provide backpressure since parsed/embedded batches are
memory-heavy (source text, then float vectors). The caller must release the GIL
before entering `.run()`; each thread re-acquires the GIL independently via
`Python::with_gil()`. The store thread reuses the existing HNSW "bulk mode"
bracket (`drop_all_hnsw_indexes()` → N incremental writes →

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chunkhound/chunkhound](https://github.com/chunkhound/chunkhound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

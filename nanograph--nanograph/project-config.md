---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is nanograph?

Embedded local-first typed property graph DB in Rust. Arrow-native columnar execution, Lance storage, DataFusion query engine. Think SQLite for graphs. Two custom DSLs: schema (.pg) and query (.gq), both parsed with Pest grammars.

## Build & Test Commands

```bash
cargo build                              # full workspace build
cargo build -p nanograph                 # library only
cargo build -p nanograph-cli             # CLI only
cargo test                               # all tests (unit + e2e + migration)
cargo test -p nanograph                  # library tests only
cargo test -p nanograph --test engine_integration  # engine integration tests
cargo test -p nanograph --test schema_migration    # migration tests only
cargo test test_bind_by_property         # single test by name
cargo test -- --nocapture                # show stdout
cargo test -p nanograph-cli               # CLI integration tests
cargo test -p nanograph-cli --test starwars_workflows  # single CLI test file
cargo clippy                             # lint
cargo fmt                                # format
RUST_LOG=debug cargo run -p nanograph-cli -- run ...  # enable tracing
cargo bench -p nanograph                 # all criterion benchmarks
cargo bench -p nanograph --bench traversal  # single benchmark domain
NANOGRAPH_BENCH_SMOKE=1 cargo bench -p nanograph  # quick smoke mode
cargo build -p nanograph-ffi             # FFI crate (not in default-members)
cargo build -p nanograph-ts              # TS SDK crate (not in default-members)
```

Workspace `default-members` = `nanograph` + `nanograph-cli`. Plain `cargo build`/`cargo test` skip the FFI and TS SDK crates — target them explicitly with `-p`.

**Requires `protoc`** (Protocol Buffers compiler) at build time for the Lance dependency. MSRV 1.91, Rust edition 2024. `debug = 0` in dev profile (no debuginfo — builds are faster but backtraces are address-only). Dependencies are compiled with `opt-level = 2` even in dev profile so tests run at reasonable speed while the nanograph crate itself stays unoptimized for fast rebuilds. Release profile uses `lto = "thin"` and `codegen-units = 16`.

## Architecture

### Workspace

Four crates:
- `nanograph` — core library. All domain logic lives here.
- `nanograph-cli` — binary named `nanograph`. Thin clap wrapper that calls library functions.
- `nanograph-ffi` — C ABI (`cdylib`/`staticlib`) for Swift and native clients. Exports `nanograph_db_*` C functions; results are heap-allocated JSON strings freed with `nanograph_string_free`. Includes Swift Package wrapper in `swift/`.
- `nanograph-ts` — TypeScript/Node.js SDK via napi-rs. npm package `nanograph-db`. `JsDatabase` wraps core `Database` behind `Arc<tokio::sync::Mutex>`. Platform-aware `.node` loader for macOS/Linux/Windows.

All SDK crates call the same core pipeline (parse → typecheck → lower → execute → serialize via `json_output`). No logic duplication.

### Dual-Mode Execution

The system supports two execution modes that affect many code paths:
- **DB mode** (`--db path.nano`): Lance-backed persistence, supports mutations, CDC, migration, maintenance commands.
- **Legacy mode** (`--schema`/`--data` flags): In-memory GraphStorage, read-only queries. Useful for quick checks without a DB.

### Query Execution Pipeline

```
.gq text → parse_query() → QueryAST
         → typecheck_query() → TypeContext (validates against catalog)
         → lower_query() → QueryIR (pipeline of operators)
         → build_physical_plan() → DataFusion ExecutionPlan
         → execute_query() → Vec<RecordBatch>
```

### Search & Embeddings

Three search subsystems:

**Full-text search** — `@index` on String properties creates Lance inverted indexes (built automatically during load/mutation via `rebuild_node_text_indexes`). Text predicates: `search(string_prop, query)` for token-based keyword match, `fuzzy(string_prop, query[, max_edits])` for approximate match, `match_text(string_prop, query)` for contiguous phrase match. Ordering/ranking: `bm25(string_prop, query)` for lexical relevance. Results are cached per session via `TextSearchCache` in `store/runtime.rs` (keyed by type, property, query, kind, dataset version).

**Vector search** — `Vector(dim)` properties with Lance exact KNN. Two workflows:
- **Manual vectors**: Put vectors directly in JSONL data, query with `nearest(prop, $param)` ordering.
- **Auto-embedding**: Annotate a `Vector(dim)` property with `@embed(source_prop)` — embeddings are generated from the source String property at load time via OpenAI API.

**Hybrid** — `rrf(nearest(...), bm25(...))` for reciprocal rank fusion. `nearest` and `rrf` require a `limit` clause.

Embedding cache: `_embedding_cache.jsonl` in the DB directory caches content-hashed embeddings to avoid re-embedding unchanged data. Large text (>1500 chars by default) is chunked with overlap and averaged.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanograph/nanograph](https://github.com/nanograph/nanograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

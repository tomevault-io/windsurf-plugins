---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

```sh
cargo build                          # slim (default): structural index + call graph + MCP
cargo build --features semantic      # + embedding search via fastembed/ONNX (~30MB larger)
cargo test                           # all integration tests (tests/) + inline unit tests
cargo test resolver_tiers            # run a single test file
cargo test -- --test-threads=1       # if tests conflict on shared DB state
cargo clippy --all-targets -- -D warnings            # lint (slim)
cargo clippy --all-targets --features semantic -- -D warnings  # lint (semantic)
cargo fmt --check                    # format check
```

Smoke test after changes:
```sh
cargo run -- index && cargo run -- stats && cargo run -- map --json
```

## Eval (resolver accuracy)

The resolver's precision/recall is measured against SCIP (compiler-grade) ground truth. Prereqs: `scip`, `scip-go`, `scip-typescript`, `scip-python`, `rust-analyzer` on PATH.

```sh
cd eval && ./run.sh      # clones 5 corpora, builds SCIP ground truth, runs compare.py
```

Results in `eval/RESULTS.md`. All corpora must stay ≥98.5% gated precision — regressions here are release-blockers. `compare.py` enforces this with `sys.exit(1)` on violation. A corpus may get a lower `CORPUS_GATE_FLOOR` override only when RESULTS.md documents a diagnosed, structurally-permanent extractor blind spot (e.g. hono's dynamic method dispatch) — never to paper over an undiagnosed regression.

## Architecture

**Single binary CLI + library crate.** The binary (`src/main.rs`) is a thin clap dispatcher (~500 lines: clap struct definitions + dispatch match + tracing init). All logic lives in the library (`src/lib.rs` re-exports).

### CLI commands (`src/commands/` — extracted from main.rs)

Command handlers are organized into focused modules:
- `index.rs` — `cmd_index`, `cmd_reset`, workspace safety, git diff changeset
- `query.rs` — `cmd_callers`, `cmd_impact`, `cmd_where`, `cmd_context`, `cmd_search`
- `inspect.rs` — `cmd_stats`, `cmd_map`, `cmd_config_show`, daemon state view
- `proxy.rs` — `cmd_proxy`, `cmd_gain`, `cmd_rewrite`
- `hooks.rs` — `cmd_hook` (pre-edit, post-edit, proxy-rewrite)
- `daemon.rs` — `cmd_daemon`, `cmd_mcp`, `cmd_watch`
- `attach.rs` — `cmd_attach`, `cmd_setup`, `cmd_skill_manifest`, `cmd_ingest`

### Index pipeline (`src/indexer/pipeline.rs` — the orchestrator)

`IndexPipeline::run_structural()` is the hot path:
1. **Walk** (`walker.rs`) — `ignore`-crate respects `.gitignore`
2. **Hash-check** — blake3 content hash; skip unchanged files
3. **Parallel parse** (rayon) — tree-sitter AST per file, extract chunks + symbols + raw refs + complexity. CPU-bound, no DB access. Machine-generated files (`parser::is_generated` — `DO NOT EDIT`/`@generated` markers) keep symbols+chunks but skip ref extraction unless `--index-generated`
4. **Git log** — parsed in a background thread concurrently with step 3
5. **Batch insert** — single transaction, prepared statements
6. **Resolve import bindings** — module-path → file_id mapping per language, then barrel/`pub use` chain chasing (bounded fixpoint, 5 hops)
7. **Batch resolve refs** — the tiered SQL resolver (see below)
8. **Temporal metrics** — hotspot scores, change coupling from git data

### Reference resolver (the core — `pipeline.rs::batch_resolve_refs`)

Tiered SQL UPDATE cascade, each level only touches `confidence = 0.0` (unresolved) refs:

| Level | Strategy | Confidence |
|-------|----------|-----------|
| L0 | Receiver-type → `symbols.scope` exact match (unique file) | 0.95 |
| L0c | Receiver-type → type's import binding target file | 0.95 |
| L0d | Receiver-type → type's unique defining file | 0.95 |
| L0b | Named-import binding (`import { x } from './m'`) | 0.95 |
| L1 | Same-file, bare/self-like calls only | 1.0 |
| L2 | Same package/directory, unique candidate | 0.95 |
| L3 | Import-qualifier match, unique candidate | 0.9 |
| L4 | Globally unique symbol | 0.8 |
| L5 | Same package, multiple candidates (demoted) | 0.6 |
| L6 | Same-file fallback for qualified/ambiguous | 0.6/0.5 |

Ordering matters: L0 runs before L1 because type evidence beats proximity. Each tier's confidence threshold was set by measuring precision against the SCIP eval corpora.

### Language extractors (`src/graph/languages/`)

Each language implements `LanguageRefExtractor` trait: `extract()` walks the tree-sitter AST and emits `RawReference` structs with optional `receiver_type` (lite inference — method receivers, typed params, local declarations with constructor initializers; no data-flow). Extractors also handle `is_test_file()` and `resolve_import()`.

- `go.rs` — Go calls, type refs, imports
- `typescript.rs` — TS/TSX/JS (shared extractor)
- `python.rs` — Python calls, type refs, `from X import Y`
- `rust_lang.rs` — Rust calls, `use` paths, `self`/`Self` receiver inference
- `java.rs` — Java calls, type refs, `import` statements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [telagod/abyss](https://github.com/telagod/abyss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->

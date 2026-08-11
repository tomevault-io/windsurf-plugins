---
trigger: always_on
description: > This file is a navigation guide for coding agents (such as Claude/Sisyphus) working on this repository. Read it first, then consult `docs/phase.md` for per-phase tasks.
---

# AGENTS.md — Navigation for Coding Agents

> This file is a navigation guide for coding agents (such as Claude/Sisyphus) working on this repository. Read it first, then consult `docs/phase.md` for per-phase tasks.

## What is docq

`docq` is a **local-first, full RAG system** (retrieval + answer synthesis) written in Rust as a library-first multi-crate workspace:
- Millisecond-latency `search` for agents (BM25 + dense vectors + RRF + cross-encoder rerank), with zero LLM cost.
- `ask` for humans — natural-language answers with inline `[N]` citations, running on a local GGUF model or an OpenAI-compatible endpoint.
- All indexes live in a single SQLite file (`sqlite-vec` + `FTS5` + plain tables); the whole pipeline works offline.
- Chinese-optimized: chunking replicates LlamaIndex's `SentenceSplitter`; BM25 uses jieba word-level pre-tokenization.

See `README.md` for the vision, `docs/design.md` for the authoritative design, and `docs/phase.md` for the phased task list.

## Current progress

See the overview table at the top of `docs/phase.md` (each phase has ✅ / ⬜). Current state:

| Phase | Status |
|---|---|
| P0 technical validation | ✅ done (verified by user; POC code not in repo) |
| P1 Workspace + Core types/traits | ✅ done |
| P2 Storage trait + SQLite basics | ✅ done (no InMemoryStorage — removed) |
| P3.1 sqlite-vec integration | ✅ done |
| P3.2 FTS5 integration | ✅ done |
| P3.3 Transactional consistency (StorageTx) | ✅ done |
| P12 CLI | ✅ done |
| P13 | ⬜ pending |

## Common commands

```bash
# Fastest compile check
cargo check --workspace

# Run all tests
cargo test --workspace

# Run tests for one crate
cargo test -p docq-storage

# Format check and apply
cargo fmt --all -- --check
cargo fmt --all

# Clippy (project requires -D warnings)
cargo clippy --all-features -- -D warnings
```

**Notes:**
- The Rust toolchain is pinned to `1.95.0` stable in `rust-toolchain.toml`. Do not use nightly.
- On macOS, `llama-cpp-2` defaults to Metal. This project forces the CPU backend via `GGML_METAL=OFF` in `.cargo/config.toml` for macOS 14.x compatibility — do not remove this setting.
- The first `cargo check` builds heavy C/C++ dependencies (`libsqlite3-sys`, `llama-cpp-sys-2`, `ort-sys`); 5–10 minutes is normal. Incremental builds are much faster.

## Architecture and crate relationships

```
        cli (docq)        mcp (future)
         └───────┬─────────┘
                 ▼
            docq            facade — the only crate library users touch
          ╱      │      ╲
  retrieve     index    synthesize        synthesize is optional (feature "ask")
      │  ╲      │          │
      │   ╲     │          ▼
      │    ╲    │       model           GGUF / ONNX backends
      │     ╲   │      ╱
      ▼      ▼ ▼     ▼
           core                     types + traits, zero heavy dependencies
```

### Crate responsibilities and dependencies

| crate | responsibility | depends on |
|---|---|---|
| `docq-core` | All core types + traits + error types; zero internal deps | none |
| `docq-model` | Model registry, HF download cache, verification, inference backends (Embedder/Reranker/Llm) | core |
| `docq-indexer` | File reading, chunking, incremental indexing, content-addressed dedup | core + storage + model(embed) |
| `docq-storage` | SQLite `Storage` impl: documents / chunks / `vec_chunks` (sqlite-vec) / `fts_chunks` (FTS5) / model_versions | core |
| `docq-retrieve` | BM25 + vector recall → RRF fusion → rerank; returns `SearchHit` + `ScoreExplain` | core + storage + model(rerank) |
| `docq-synth` | Ask: build prompt → LLM → parse `[N]` citations → `Answer` | core + retrieve + model(llm) |
| `docq` | CLI binary (currently empty shell, implemented in P12) | all of the above |

### Layering rules (important — do not break)

- Upper layers may depend on lower layers; **lower layers must not depend on upper layers**.
- `docq-core` does not depend on any other internal crate — it defines all traits that other crates implement. This is the key to the library-first promise: `cargo add docq-core` does not pull in SQLite, llama.cpp, or other heavy stacks.
- `indexer` and `retrieve` do not depend on each other; both operate on data via the `Storage` trait.
- SQLite details are fully isolated within `docq-storage`.
- `docq-model` uses feature flags so consumers enable only the backends they need (avoiding "just want search but must compile llama.cpp").

## Key design decisions (do not overturn without intent)

1. **The `Storage` trait stays in `docq-core`**, not in `docq-storage`. This is dependency inversion: `indexer` and `retrieve` only need `docq-core` + `docq-model` and do not pull `rusqlite`/`sqlite-vec` at compile time. A future `docq-storage-pg` would be a drop-in replacement.
2. **No `InMemoryStorage`**. Tests use `SqliteStorage::open_in_memory()` (SQLite `:memory:` mode, millisecond startup).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lichuang/docq](https://github.com/lichuang/docq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

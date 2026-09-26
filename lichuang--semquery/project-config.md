---
trigger: always_on
description: > This file is a navigation guide for coding agents working on this repository. Read it first.
---

# AGENTS.md — Navigation for Coding Agents

> This file is a navigation guide for coding agents working on this repository. Read it first.

## What is semquery

`semquery` is a **local-first, full RAG system** (retrieval + answer synthesis) written in Rust as a library-first multi-crate workspace:
- Millisecond-latency `search` for agents (BM25 + dense vectors + RRF + cross-encoder rerank), with zero LLM cost.
- `ask` for humans — natural-language answers with inline `[N]` citations, running on a local GGUF model or an OpenAI-compatible endpoint.
- All indexes live in a single SQLite file (`sqlite-vec` + `FTS5` + plain tables); the whole pipeline works offline.
- Chinese-optimized: chunking replicates LlamaIndex's `SentenceSplitter`; BM25 uses jieba word-level pre-tokenization.

See `README.md` for the project overview.

## Naming convention

The project uses two names with distinct roles:

- **`semquery`** = project / package name (developer layer, `cargo add semquery`).
- **`semq`** = command / runtime name (user layer, the command you type and the directories you see).

So the project is named `semquery`, the CLI command is `semq`. Sub-crates keep the `semquery-*` prefix (`semquery-core`, `semquery-model`, ...) following the Rust convention of prefixing sub-crates with the main crate name.

Runtime artifacts (config dir, cache dir, DB file, log file) use `semq` to match the command name:
- `~/.config/semq`
- `~/.cache/semq/models`
- `semq.db`
- `semq.log`

## Common commands

```bash
# Fastest compile check
cargo check --workspace

# Run all tests
cargo test --workspace

# Run tests for one crate
cargo test -p semquery-storage

# Format check and apply
cargo fmt --all -- --check
cargo fmt --all

# Clippy (project requires -D warnings)
cargo clippy --all-features -- -D warnings

# Run the full pre-commit suite (check + test + fmt + clippy) in one shot
./pre-commit-check.sh
```

**Notes:**
- The Rust toolchain is pinned to `1.95.0` stable in `rust-toolchain.toml`. Do not use nightly.
- On macOS, `llama-cpp-2` defaults to Metal. This project enables Metal via `GGML_METAL=ON` in `.cargo/config.toml` (for both `aarch64-apple-darwin` and `x86_64-apple-darwin`), plus `CMAKE_CXX_FLAGS="-Wno-elaborated-enum-base"` to silence a warning. Do not remove these settings.
- The first `cargo check` builds heavy C/C++ dependencies (`libsqlite3-sys`, `llama-cpp-sys-2`, `ort-sys`); 5–10 minutes is normal. Incremental builds are much faster.

## Architecture and crate relationships

```
        cli (semquery)        mcp (future)
         └───────┬─────────┘
                 ▼
            semquery            facade — the only crate library users touch
          ╱      │      ╲
  retrieve     index    synthesize        synthesize is optional (needs semquery-model/llm)
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
| `semquery-core` | All core types + traits + error types; zero internal deps | none |
| `semquery-model` | Model registry, HF download cache, verification, inference backends (Embedder/Reranker/Llm) | core |
| `semquery-indexer` | File reading, chunking, incremental indexing, content-addressed dedup | core + storage + model(embed) |
| `semquery-storage` | SQLite `Storage` impl: documents / chunks / `vec_chunks` (sqlite-vec) / `fts_chunks` (FTS5) / model_versions | core |
| `semquery-retrieve` | BM25 + vector recall → RRF fusion → rerank; returns `SearchHit` + `ScoreExplain` | core + storage + model(rerank) |
| `semquery-synth` | Ask: build prompt → LLM → parse `[N]` citations → `Answer` | core + retrieve + model(llm) |
| `semquery` | CLI binary and `Engine` facade; exposes `init/add/index/search/ask/status` plus `Engine` for library users | all of the above |

### Layering rules (important — do not break)

- Upper layers may depend on lower layers; **lower layers must not depend on upper layers**.
- `semquery-core` does not depend on any other internal crate — it defines all traits that other crates implement. This is the key to the library-first promise: `cargo add semquery-core` does not pull in SQLite, llama.cpp, or other heavy stacks.
- `indexer` and `retrieve` do not depend on each other; both operate on data via the `Storage` trait.
- SQLite details are fully isolated within `semquery-storage`.
- `semquery-model` uses feature flags (`embed` / `rerank` / `llm`, all on by default) so consumers enable only the backends they need (avoiding "just want search but must compile llama.cpp").

## Key design decisions (do not overturn without intent)

1. **The `Storage` trait stays in `semquery-core`**, not in `semquery-storage`. This is dependency inversion: `indexer` and `retrieve` only need `semquery-core` + `semquery-model` and do not pull `rusqlite`/`sqlite-vec` at compile time. A future `semquery-storage-pg` would be a drop-in replacement.
2. **No `InMemoryStorage`**. Tests use `SqliteStorage::open_in_memory()` (SQLite `:memory:` mode, millisecond startup).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lichuang/semquery](https://github.com/lichuang/semquery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

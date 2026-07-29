---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

SQLRite is a from-scratch SQLite-style embedded database written in Rust. It's published on crates.io as `sqlrite-engine` (imported as `use sqlrite::…` — the lib target keeps the short name) and ships as: a REPL binary (`sqlrite`), a Tauri 2 + Svelte 5 desktop app, a Model Context Protocol stdio server (`sqlrite-mcp`), a C FFI shim (`sqlrite-ffi`), and language SDKs (Python via PyO3, Node via napi-rs, Go via cgo, WASM via wasm-bindgen). Phases 1–7 are shipped; the current branch `phase-8-plan` drafts inverted-index + BM25 full-text search and hybrid retrieval.

## Workspace layout

`Cargo.toml` is a workspace whose members are: `.` (the engine, package `sqlrite-engine`, lib `sqlrite`), `desktop/src-tauri`, `sqlrite-ffi`, `sqlrite-ask`, `sqlrite-mcp`, `sdk/python`, `sdk/nodejs`, `benchmarks`. `sdk/wasm` and `sdk/go` are deliberately **not** workspace members (wasm32 target / cgo separation).

- `src/` — engine. Public API is `Connection`/`Statement`/`Rows`/`Row`/`Value` from [src/connection.rs](src/connection.rs), re-exported via [src/lib.rs](src/lib.rs). Any new SDK should bind only to this surface.
- `sqlrite-ask/` — pure-Rust LLM adapter (Anthropic/OpenAI/Ollama) for natural-language → SQL. The engine's `ask` feature provides the thin `ConnectionAskExt::ask` glue.
- `sqlrite-mcp/` — MCP stdio server. Seven tools: `list_tables`, `describe_table`, `query`, `execute`, `schema_dump`, `vector_search`, `ask`. `--read-only` opens with a shared lock and hides `execute`.
- `sqlrite-ffi/` — C ABI cdylib + generated `sqlrite.h` header. Backs the Go SDK and any C consumer.
- `desktop/` — Tauri 2 + Svelte 5 GUI. Embeds the engine directly (no FFI hop).
- `benchmarks/` — SQLR-4 / SQLR-16 bench harness. `Driver` trait + SQLRite + SQLite (rusqlite-bundled) drivers + criterion-driven workloads. Excluded from the default CI build/test/clippy/doc commands; run locally with `make bench` (or `make bench-duckdb`). See [docs/benchmarks-plan.md](docs/benchmarks-plan.md).
- `web/` — marketing + docs site (Next.js 15 + Tailwind v4). Independent of the Cargo workspace; lives in-repo for now but is structured to lift into its own repository later. See [web/README.md](web/README.md).

Architecture deep-dive: [docs/architecture.md](docs/architecture.md). The full doc index is [docs/_index.md](docs/_index.md).

## Engine data flow

SQL string → [src/sql/mod.rs](src/sql/mod.rs) `process_command` parses with the external `sqlparser` crate (SQLite dialect) → [src/sql/parser/](src/sql/parser/) trims the AST into internal structs (`CreateQuery`, `InsertQuery`, `SelectQuery`) → [src/sql/executor.rs](src/sql/executor.rs) runs the statement against the in-memory `Database` ([src/sql/db/database.rs](src/sql/db/database.rs)). On any write, auto-save serializes changed pages through [src/sql/pager/](src/sql/pager/) — 4 KiB pages, cell-encoded B-trees per table and index, WAL + crash-safe checkpoint, fs2 advisory locks. Vector search (Phase 7d) goes through [src/sql/hnsw.rs](src/sql/hnsw.rs); KNN uses a bounded-heap top-k in the executor. Transactions snapshot the in-memory state; ROLLBACK restores it. There is no query optimizer beyond the KNN/HNSW shortcut, no joins, no aggregates yet.

## Commands

CI is the source of truth — the workspace excludes that follow are required because the desktop crate needs a Svelte build first, the PyO3/napi-rs cdylibs can't link standalone test binaries, and the `benchmarks/` harness deliberately stays out of CI (criterion is noisy on shared runners; the rusqlite-bundled build is heavy).

```sh
# Build / test the Rust workspace (matches CI)
cargo build --workspace --exclude sqlrite-desktop --exclude sqlrite-python --exclude sqlrite-nodejs --exclude sqlrite-benchmarks --all-targets
cargo test  --workspace --exclude sqlrite-desktop --exclude sqlrite-python --exclude sqlrite-nodejs --exclude sqlrite-benchmarks

# Single test (exact name; --nocapture to see println!)
cargo test <test_name> -- --nocapture

# Lint (CI runs all three)
cargo fmt --all -- --check
cargo clippy --workspace --exclude sqlrite-desktop --exclude sqlrite-python --exclude sqlrite-nodejs --exclude sqlrite-benchmarks --all-targets
cargo doc    --workspace --exclude sqlrite-desktop --exclude sqlrite-python --exclude sqlrite-nodejs --exclude sqlrite-benchmarks --no-deps

# Run the REPL (default features include cli + ask + file-locks)
cargo run                                  # in-memory
cargo run -- path/to/db.sqlrite            # open/create file
cargo run -- --readonly path/to/db.sqlrite # shared-lock open

# Crate-specific
cargo build --release -p sqlrite-ffi       # C cdylib + sqlrite.h
cd desktop && npm install && npm run tauri dev   # desktop app dev mode
cargo run -p sqlrite-mcp -- /path/to.sqlrite     # MCP server (stdio)

# Benchmarks (SQLR-4 / SQLR-16) — local-only, never CI
make bench                                 # SQLRite + SQLite (lean)
make bench-duckdb                          # adds DuckDB driver (Group B only)

# Release plumbing
scripts/bump-version.sh 0.2.0              # bumps version across 11 manifests
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joaoh82/rust_sqlite](https://github.com/joaoh82/rust_sqlite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

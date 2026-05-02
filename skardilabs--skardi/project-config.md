---
trigger: always_on
description: This file gives AI agents and human contributors the minimum context needed to make safe, useful changes to Skardi. Read this before opening a PR.
---

# AGENTS.md

This file gives AI agents and human contributors the minimum context needed to make safe, useful changes to Skardi. Read this before opening a PR.

Skardi is a federated SQL query engine and YAML-driven REST API server, written in Rust on top of Apache DataFusion. The CLI runs SQL against local files, object stores, databases, and lakehouse formats; the server turns SQL pipelines defined in YAML into parameterized HTTP endpoints — zero application code required.

Project-specific Claude Code instructions also live in [.claude/CLAUDE.md](.claude/CLAUDE.md). When the two files disagree, CLAUDE.md wins.

## Repository Layout

Cargo workspace with three crates:

- [crates/skardi/](crates/skardi/) — the core library: query engine, pipelines, model inference, and all data source providers. Everything reusable lives here.
  - `engine/` — DataFusion engine wrapper.
  - `pipeline/` — YAML pipeline parsing, parameter inference, schema inference.
  - `model/` — embedding/ONNX model registries (`candle`, `gguf`, `onnx`).
  - `sources/providers/` — one subdirectory per backend: `lance/`, `iceberg.rs`, `mongo/`, `redis/`, `sqlite/`, `sqlx/pg/`, `mysql.rs`, etc.
- [crates/server/](crates/server/) — the `skardi-server` binary: Axum HTTP server, auth (better-auth), config loading, metrics, telemetry. Wires the core library into a long-running service.
- [crates/cli/](crates/cli/) — the `skardi` CLI binary. Thin wrapper around the core library for one-shot SQL execution.

Other top-level directories: [docs/](docs/) (per-source guides + `server.md`), [demo/](demo/) (end-to-end example apps), [asset/](asset/) (logo, badges, architecture diagram), [data/](data/) (sample data for tests/demos).

## Build, Test, Lint

```bash
# Check the entire workspace (fast feedback)
cargo check --workspace --all-targets

# Build
cargo build --release -p skardi-cli
cargo build --release -p skardi-server

# Optional embedding stack (ONNX, GGUF, Candle, remote embed)
cargo build --release -p skardi-server --features embedding

# Tests
cargo test --workspace
cargo test -p skardi <test_name>

# Format and lint (also enforced by pre-commit hooks)
cargo fmt --all
cargo clippy --workspace --all-targets
```

Pre-commit hooks (`.pre-commit-config.yaml`) run `cargo fmt` and `cargo check --all` on every commit. Don't bypass them with `--no-verify` — fix the underlying issue.

## Coding Standards

### Imports

- Always import types at the top of the file with `use` statements. **Never use full crate paths inline in function bodies.** This is enforced by code review.

```rust
// GOOD
use std::sync::Arc;
use datafusion::prelude::SessionContext;

fn make_ctx() -> Arc<SessionContext> { Arc::new(SessionContext::new()) }

// BAD
fn make_ctx() -> std::sync::Arc<datafusion::prelude::SessionContext> {
    std::sync::Arc::new(datafusion::prelude::SessionContext::new())
}
```

### Error Handling — No Panics in Production Code

**No raw `.unwrap()` is allowed anywhere outside `crates/cli/` and test code** (`#[cfg(test)]` modules, `#[test]` functions, files under `tests/`, doc-comment examples). Pick the strategy that matches the failure mode:

1. **Recoverable errors** → propagate via `Result` with `?`, `ok_or_else`, `with_context`, or a custom error variant. The library uses `anyhow::Result` for application code and `thiserror`-style enums (e.g. `ConfigError`, `DataFusionError::Internal(...)`) at module boundaries. Add a new error variant rather than stuffing context into a generic string.
2. **Lock poisoning** on `std::sync::RwLock`/`Mutex` → recover with `.unwrap_or_else(|p| p.into_inner())` so a poisoned lock degrades gracefully instead of panicking. Skardi uses this pattern in the optimizer registry, model caches, and Lance dataset registry.
3. **True invariants** that cannot fail at runtime → use `.expect("why this cannot fail")` with a message documenting the invariant. Examples: `"len == 1 checked above"`, `"DataType::List guarantees ListArray"`. `expect` still panics, so reserve it for cases the type system or a prior check has already ruled out.

`.unwrap()` is fine in CLI code, tests, and doc examples. Production library and server code should have none.

### Logging

- Use `tracing::{info, warn, error, debug}` — never the `log` crate, never `println!` / `dbg!` in committed code.
- Keep log messages on a single line. Include identifiers (data source name, pipeline name, table name) so logs are filterable.

### Async

- The runtime is Tokio. Don't block in async functions — use `tokio::fs`, `tokio::time::sleep`, and `tokio::task::spawn_blocking` for sync I/O.
- Don't hold a `std::sync::Mutex` / `RwLock` guard across an `.await`. If you need to hold state across awaits, restructure or use `tokio::sync::Mutex`.

### Adding a Data Source

Most providers follow the same shape — use an existing one (e.g. [crates/skardi/src/sources/providers/sqlite/](crates/skardi/src/sources/providers/sqlite/) or [crates/skardi/src/sources/providers/lance/](crates/skardi/src/sources/providers/lance/)) as a template:

1. Add a module under `crates/skardi/src/sources/providers/<name>/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SkardiLabs/skardi](https://github.com/SkardiLabs/skardi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->

---
trigger: always_on
description: Cognee-Rust is a Rust port of the Python [cognee](https://github.com/topoteretes/cognee) library — an AI memory pipeline that transforms raw data into persistent, queryable knowledge graphs. The goal is both to run on edge devices (Android, embedded) with local models and to serve as a drop-in replacement of the Python `cognee` SDK, while maintaining 90%+ correctness parity.
---

# Cognee-Rust Project Guide

## Project Overview

Cognee-Rust is a Rust port of the Python [cognee](https://github.com/topoteretes/cognee) library — an AI memory pipeline that transforms raw data into persistent, queryable knowledge graphs. The goal is both to run on edge devices (Android, embedded) with local models and to serve as a drop-in replacement of the Python `cognee` SDK, while maintaining 90%+ correctness parity.

**Core pipeline:** `add (ingest)` → `cognify (knowledge graph extraction)` → `search (context retrieval)`

## Python Reference Codebase

The Python implementation in the [cognee repository](https://github.com/topoteretes/cognee) (under the `cognee/` directory) serves as the reference for all Rust ports. If you need the Python sources for reference, clone with `git clone --depth 1 https://github.com/topoteretes/cognee.git /tmp/cognee-python`. If the task requires understanding of the Python codebase, read the documentation in that repository (e.g. `/tmp/cognee-python/README.md`, docs, and inline docstrings) before proceeding.

## Workspace structure, crates, patterns & dependencies

The workspace layout, the crate-by-crate breakdown, the cross-cutting
architecture patterns, the key-dependency table, and the rustdoc guide live in
the public docs — single source of truth:

- **[docs/architecture.md](../docs/architecture.md)** — workspace tree, crate
  breakdown, architecture patterns, key dependencies, `cargo doc` guide.
- **[docs/operations.md](../docs/operations.md)** — what `add`/`cognify`/`memify`/`search` and the lifecycle ops do.
- **[docs/configuration.md](../docs/configuration.md)** — full env-var / `Settings` / `ConfigManager` reference.
- **[docs/tools/](../docs/tools/README.md)** — CLI, bindings, HTTP server, pluggable backends.
- **[docs/README.md](../docs/README.md)** — documentation hub / index.

Keep `docs/architecture.md` updated when crates are added or change — do not
re-introduce a duplicate crate list here.

## Build & Development

```bash
# Format the code
cargo fmt

# Check compilation (all targets including tests and examples)
cargo check --all-targets

# Run clippy
cargo clippy --all-targets

# Run tests (debug mode by default, no --release unless explicitly asked)
cargo test

# After making changes, run the full check suite:
scripts/check_all.sh
```

### Cleaning build artifacts (disk space)

The repo has **three** Cargo workspaces (root, `capi/`, `bindings/`), so `cargo clean` at
the root only clears one of them (`bindings/target` alone routinely exceeds 20 GB). Never
hand-roll `rm -rf` over target dirs — use `bash scripts/clean_all.sh` (`--help` lists the
flags, `--dry-run` reports sizes without deleting).

Do not restate the flag/artifact list here or in the READMEs — it drifts. The canonical
prose lives in
[CONTRIBUTING.md § Cleaning build artifacts](../CONTRIBUTING.md#cleaning-build-artifacts)
and the runtime source of truth is the script's own `--help`. When a new Cargo workspace or
generated artifact dir is added, extend `CARGO_MANIFESTS` / `ALL_ARTIFACTS` in the script
(and its header comment, which is what `--help` prints).

## Test Patterns

- **Async tests:** `#[tokio::test]` for all async test functions (only async runtime used)
- **Mock objects** (behind `testing` feature flag): `MockStorage` (HashMap-based), `MockGraphDB`, `MockVectorDB`. No MockDatabase — tests use real in-memory SQLite (`sqlite::memory:`). All mocks re-exported via `cognee-test-utils`.
- **Temp directories:** `tempfile::tempdir()` for isolated test environments
- **Inline tests:** `#[cfg(test)] mod tests` in source files for focused unit tests
- **Integration tests:** 27 files under `crates/*/tests/` across 12 crates (ingestion, cognify, search, database, embedding, session, CLI, etc.)
- **E2E tests:** CLI E2E via `assert_cmd`, integration tests requiring `COGNEE_E2E_EMBED_MODEL_PATH` / `COGNEE_E2E_TOKENIZER_PATH` env vars, cross-SDK tests in `e2e-cross-sdk/`
- **Conditional skipping:** Tests gracefully skip when required env vars or models are unavailable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [topoteretes/cognee-rs](https://github.com/topoteretes/cognee-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

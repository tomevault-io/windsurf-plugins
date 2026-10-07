---
trigger: always_on
description: Rust libraries for Xet Storage: chunk-based deduplication, uploads, and file reconstruction for the Hugging Face Hub. `hf_xet` exposes them to Python through `huggingface_hub`.
---

# Agent Guide for xet-core

## Project overview

Rust libraries for Xet Storage: chunk-based deduplication, uploads, and file reconstruction for the Hugging Face Hub. `hf_xet` exposes them to Python through `huggingface_hub`.

## Architecture documentation

**Read the documentation and implementation for the layer you are changing:**

- [Project overview](README.md) — setup and usage.
- [Session API](xet_pkg/README.md) — public Rust API and crate relationships.
- [Data processing](xet_data/README.md) — chunking, deduplication, and reconstruction.
- [Client](xet_client/README.md) — CAS and Hub communication.
- [Core structures](xet_core_structures/README.md) — hashes, shards, and Xorb objects.
- [Runtime](xet_runtime/README.md) — async runtime, configuration, and logging.
- [Python extension](hf_xet/README.md) — hf-xet Python package.
- [Git extension](git_xet/README.md) — Git LFS transfer agent.
- [Browser WASM wrapper](wasm/hf_xet_wasm/README.md) — upload/download WASM APIs and smoke tests; an example, not a published SDK.
- [Thin WASM extension](wasm/hf_xet_thin_wasm/README.md) — chunking and hashing for JavaScript; [benchmarks](wasm/hf_xet_thin_wasm/CHUNKER_BENCHMARK.md).

`CLAUDE.md` links to this file. Keep shared agent guidance here. Component-specific build and test instructions live in [hf_xet/AGENTS.md](hf_xet/AGENTS.md), [git_xet/AGENTS.md](git_xet/AGENTS.md), and [wasm/AGENTS.md](wasm/AGENTS.md).

## Setup

- **Rust**: follow the version pinned in [CI](.github/workflows/ci.yml). Use nightly `rustfmt`.
- **Python**: activate a virtualenv and install `maturin` and `pytest`. Build/install with `maturin develop`; produce wheels with `maturin build`.
- **Git tests**: install Git LFS and run `git lfs install`.
- **Separate manifests**: `hf_xet`, `simulation`, `wasm/*`, and `examples/xet_pkg_napi` are excluded from the root workspace. Check them separately when affected.

## Key commands

Run from the repository root unless noted.

| Command | Purpose |
| --- | --- |
| `cargo build` | Build the root workspace |
| `cargo test` | Test the root workspace with default features |
| `cargo test --package hf-xet` | Test one package; substitute the affected package |
| `cargo test --package xet-client test_name` | Run matching tests |
| `cargo build --package hf-xet --example example --features internal-tools` | Build the async example; use `example_sync` for sync |
| `cargo clippy -r --verbose -- -D warnings` | Lint the root workspace |
| `cargo clippy -r --verbose --manifest-path hf_xet/Cargo.toml -- -D warnings` | Lint Python bindings |
| `cargo +nightly fmt --manifest-path Cargo.toml --all` | Format the root workspace |
| `cargo +nightly fmt --manifest-path hf_xet/Cargo.toml --all` | Format Python bindings |
| `cargo test --manifest-path hf_xet/Cargo.toml --verbose --no-fail-fast` | Test Python bindings in Rust |
| `(cd hf_xet && maturin develop)` | Build/install the Python extension |
| `pytest hf_xet/tests/ -v` | Test the rebuilt Python extension |
| `cargo bench --no-run --workspace --exclude git_xet` | Check benchmark compilation |

Use Cargo for binding tests/lints and maturin for importable extensions. Keep PyO3's `auto-initialize` feature in test dependencies only.

The `simulation` feature enables workspace support, not the excluded `simulation/` package. The `cargo smoke-test` alias currently references that excluded package; use targeted tests instead.

## Verification

Run the checks that cover what you changed before claiming work is done. Documentation-only changes need no builds or tests. These mirror [CI](.github/workflows/ci.yml); everything here runs locally without network access or Hub credentials.

Every Rust change:

```bash
cargo +nightly fmt --manifest-path Cargo.toml --all -- --check
cargo +nightly fmt --manifest-path hf_xet/Cargo.toml --all -- --check
cargo clippy -r --verbose -- -D warnings
cargo clippy -r --verbose --manifest-path hf_xet/Cargo.toml -- -D warnings
cargo test --package <changed-package>
```

Full root workspace with CI features (required for Git, simulation, or cross-crate changes):

```bash
cargo test --verbose --no-fail-fast --features "strict simulation internal-tools git-xet-for-integration-test"
```

Per-component checks; each directory has its own `AGENTS.md` with setup details:

| Changed area | Run | Details |
| --- | --- | --- |
| `hf_xet/` or anything it depends on | `cargo test --manifest-path hf_xet/Cargo.toml --no-fail-fast`, then `(cd hf_xet && maturin develop) && pytest hf_xet/tests/ -v` | [hf_xet/AGENTS.md](hf_xet/AGENTS.md) |
| `git_xet/` | `cargo test --package git_xet --features git-xet-for-integration-test` | [git_xet/AGENTS.md](git_xet/AGENTS.md) |
| `wasm/*` or the crates they depend on | `(cd xet_pkg && ./build_wasm.sh)`, `(cd wasm/hf_xet_thin_wasm && ./build_wasm.sh)`, `(cd wasm/hf_xet_wasm && ./build_wasm.sh)` | [wasm/AGENTS.md](wasm/AGENTS.md) |
| `benches/` or bench dependencies | `cargo bench --no-run --workspace --exclude git_xet` | |
| Dependency changes | `cargo machete`, `cargo audit -D warnings`, then confirm `git status --porcelain '*Cargo.lock'` is empty | |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/xet-core](https://github.com/huggingface/xet-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

---
trigger: always_on
description: Delta-kernel-rs is a Rust library for building Delta Lake connectors. It encapsulates the
---

# AGENTS.md

## Project Overview

Delta-kernel-rs is a Rust library for building Delta Lake connectors. It encapsulates the
Delta protocol so connectors can read and write Delta tables without understanding protocol
internals. Kernel never does I/O directly: it defines _what_ to do via its APIs
(`Snapshot`, `Scan`, `Transaction`) and delegates _how_ to the `Engine` trait.

Current capabilities include table reads with predicates, data skipping, deletion vectors,
change data feed, incremental scans (`incremental_scan_builder`) and commit ranges, checkpoints
(V1 & V2), version checksums, blind appends, file removals, table creation (including clustered
tables), limited schema alteration, and catalog-managed tables. Log compaction remains disabled
(#2337).

## Build & Test Commands

> **`datafusion-executor` and `integration-tests` are separate workspaces.** Root `--workspace`
> commands do not include them. For `datafusion_executor` commands, see
> `datafusion-executor/CLAUDE.md`. `integration-tests/test-all-arrow-versions.sh` tests each
> supported Arrow version.

```bash
# Build
cargo build --workspace --all-features

# Run all tests (prefer nextest over cargo test)
cargo nextest run --workspace --all-features

# Run tests for a specific crate
cargo nextest run -p delta_kernel --all-features

# Run a single test in a specific crate (fastest: only compiles that crate)
cargo nextest run -p delta_kernel --lib --all-features test_name_here

# Run a test by name, searching all crates (slow: compiles everything)
cargo nextest run --workspace --all-features test_name_here

# Format, lint, and doc check (always run after code changes)
cargo +nightly fmt \
  && cargo clippy --workspace --benches --tests --all-features -- -D warnings \
  && cargo doc --workspace --all-features --no-deps

# Split no-default-features CI checks (cargo aliases from .cargo/config.toml)
cargo clippy-no-default-kernel-dependents
cargo check-no-default-kernel
cargo check-no-default-engine
cargo clippy-no-default-kernel-leaves

# Quick pre-push check (mimics CI)
cargo +nightly fmt \
  && cargo clippy --workspace --benches --tests --all-features -- -D warnings \
  && cargo doc --workspace --all-features --no-deps \
  && cargo nextest run --workspace --all-features
```

### Crate Names for `-p` Flag

| Crate                                | Directory                             | Description                                                              |
|--------------------------------------|---------------------------------------|--------------------------------------------------------------------------|
| `delta_kernel`                       | `kernel/`                             | Core library                                                             |
| `delta_kernel_default_engine`        | `default-engine/`                     | Default Arrow/Tokio `Engine` implementation                              |
| `delta_kernel_default_engine_test_utils` | `default-engine/test-utils/`      | Default-engine test utilities                                            |
| `delta_kernel_ffi`                   | `ffi/`                                | C/C++ FFI bindings                                                       |
| `delta_kernel_ffi_macros`            | `ffi-proc-macros/`                    | FFI proc macros                                                          |
| `delta_kernel_derive`                | `derive-macros/`                      | Proc macros                                                              |
| `acceptance`                         | `acceptance/`                         | Acceptance tests (DAT)                                                   |
| `test_utils`                         | `test-utils/`                         | Shared test utilities                                                    |
| `delta_kernel_workloads`             | `workloads/`                          | Shared workload spec types + SQL predicate parser                        |
| `delta_kernel_benchmarks`            | `benchmarks/`                         | Workload benchmarks                                                      |
| `feature_tests`                      | `feature-tests/`                      | Feature flag tests                                                       |
| `mem-test`                           | `mem-test/`                           | Memory-usage test executable                                             |
| `delta-kernel-unity-catalog`         | `delta-kernel-unity-catalog/`         | Unity Catalog integration (UCCommitter, snapshot + create-table helpers) |
| `unity-catalog-delta-client-api`     | `unity-catalog-delta-client-api/`     | Transport-agnostic UC client traits + wire models                        |
| `unity-catalog-delta-rest-client`    | `unity-catalog-delta-rest-client/`    | REST/HTTP client for the Unity Catalog Delta Tables API                  |

Packages under `kernel/examples/` are also workspace members. Use the package name from the
example's `Cargo.toml` with `-p`.

### Feature Flags

Some noteworthy ones (see `[features]` in `kernel/Cargo.toml` for the full list):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [delta-io/delta-kernel-rs](https://github.com/delta-io/delta-kernel-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

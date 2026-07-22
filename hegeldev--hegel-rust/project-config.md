---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code), Codex 5.5, and other coding agents when working with code in this repository. The legacy `.claude` path is a symlink to `.agents`, and `.claude/CLAUDE.md` resolves back to this file so Claude and Codex share the same instructions.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code), Codex 5.5, and other coding agents when working with code in this repository. The legacy `.claude` path is a symlink to `.agents`, and `.claude/CLAUDE.md` resolves back to this file so Claude and Codex share the same instructions.

## Overview

This repository is the Rust implementation of Hegel, a universal property-based testing framework. It contains two crates: the native engine (a port of Hypothesis's conjecture engine) lives in the `hegel-c` workspace member and is built as the `libhegel` C-ABI library, and the `hegeltest` frontend (the root crate) drives that engine through the same C ABI every other language binding uses. Everything runs in-process — there is no external server or Python dependency.

## Build & Test Commands

```bash
just check                          # run full CI checks (lint + tests + all-features tests)
just test                           # run tests
just lint                           # run clippy, rustfmt --check, and the repo lint scripts
just format                         # format
just docs                           # build and open docs
just check-coverage                 # check coverage (requires cargo-llvm-cov + llvm-tools-preview)
just c-test                         # hegel-c smoke tests + example C programs
just miri                           # fast core of the suite under Miri
cargo test test_name                # run a single test
```

MSRV is 1.86 (enforced in CI and Cargo.toml). If you bump it, also bump `ci.yml`, `hegel-macros/Cargo.toml`, and `hegel-c/Cargo.toml`.

## Workspace Structure

### `hegeltest` (root crate) — the Rust frontend

- `src/lib.rs` — Public API surface: `hegel()`, the `Hegel` builder, `TestCase`, the `Generator` trait, and the proc-macro re-exports (`#[hegel::test]`, `#[hegel::main]`, `#[derive(DefaultGenerator)]`, `#[composite]`, `#[state_machine]`, `#[reproduce_failure]`, …)
- `src/ffi.rs` — The libhegel C-ABI boundary: the only module that touches the raw `hegel_*` functions; the rest of the frontend works against its safe wrappers (`SettingsHandle`, `RunHandle`, `CTestCase`, `RunResult`)
- `src/run_lifecycle.rs` — Cross-cutting per-test-case lifecycle: panic hook, `catch_unwind` wrapping, translating panics into `TestCaseResult`, and the final re-raise
- `src/backend.rs` — The result types the lifecycle speaks (`TestCaseResult`, `Failure`)
- `src/test_case.rs` — `TestCase` (the handle test bodies draw from) and its thread-local state, the `Collection` helper, and the span `labels` module
- `src/runner.rs` — `Hegel` builder plus `Settings`, `HealthCheck`, `Phase`, `Mode`, `Backend`, `Verbosity`
- `src/cli.rs` — CLI argument parsing for standalone `#[hegel::main]` binaries
- `src/generators/` — All first-party generator implementations (the `Generator` trait lives in `generators.rs`)
- `src/extras/` — Feature-gated third-party integrations (`chrono`, `jiff`, `serde_json`, `rand`)
- `src/stateful.rs` — Stateful (model-based) testing via `#[state_machine]`
- `src/explicit_test_case.rs` — Explicit test-case support (`#[explicit_test_case]`)
- `src/control.rs` — Control-flow unwind payloads (`AssumeFailed`, `StopTest`) and their handling
- `src/antithesis.rs` — Antithesis integration
- `hegel-macros/` — Proc-macro crate (sub-crate with its own `Cargo.toml`)

### `hegel-c` — the engine, built as `libhegel`

- `src/lib.rs` — The exported `hegel_*` C functions: settings, run lifecycle, test-case handles (including clones), draws, spans, collections, pools, state machines, targeting, results/failures. The checked-in header `hegel-c/include/hegel.h` is generated from this file by cbindgen (`just c-header`)
- `src/backend.rs` — The `DataSource` trait the engine implements and the C ABI drives
- `src/native/` — The engine proper: `core/` (choice sequence, test-case state, shrink ordering), `draws/` (the typed draw implementations: float specs, string generators, regex, internet, date/time/uuid/ip), `shrinker/`, `test_runner.rs` (owns a run: database replay, generation, targeting, shrinking, final replay), plus the failure database, data tree / novel-prefix generation, RNG, regex generation (`re/`), interval sets + Unicode tables, and blob encoding
- `src/embed.rs` — Low-level embedding entry point for driving the engine natively from Rust
- Released as `libhegel-<goos>-<goarch>.<ext>` assets on each GitHub release; the source is published to crates.io as `hegeltest-c` mostly to reserve the name

### Feature Flags (root crate)

- **`rand`**, **`chrono`**, **`jiff`**, **`serde_json`**, **`serde_json_raw_value`**: gate the corresponding `extras::` generator modules
- **`antithesis`**: Antithesis SDK integration (Linux-only; `compile_error!` on Windows)
- **`__bench`**: internal, re-exports engine internals for `benches/`; not part of the public API

## Architecture

### How It Works


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hegeldev/hegel-rust](https://github.com/hegeldev/hegel-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

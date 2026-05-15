---
trigger: always_on
description: Instructions for AI coding agents working on **SubX-CLI**.
---

# AGENTS.md

Instructions for AI coding agents working on **SubX-CLI**.

## Project Overview

SubX-CLI is an AI-powered command-line tool for automated subtitle
processing, written in Rust (edition 2024). It matches subtitle files to
videos using AI, converts between subtitle formats (SRT, ASS, VTT, SUB),
and synchronizes subtitle timing via Voice Activity Detection (VAD).

- **Repository:** <https://github.com/jim60105/subx-cli>
- **License:** GPL-3.0-or-later
- **Binary name:** `subx-cli`

## Build, Test, and Quality Commands

Trust these instructions — only search the codebase if they are incomplete
or produce errors.

| Task | Command |
|---|---|
| Build | `cargo build` |
| Release build | `cargo build --release` |
| Format | `cargo fmt` |
| Lint | `cargo clippy -- -D warnings` |
| Run tests | `cargo nextest run \|\| true` |
| **Full quality check** | `scripts/quality_check.sh` (Linux/macOS) or `scripts/quality_check.ps1` (Windows, PowerShell) |
| Full QA (verbose) | `scripts/quality_check.sh -v` or `scripts/quality_check.ps1 -VerboseOutput` |
| Coverage report | `scripts/check_coverage.sh -T` (Linux/macOS) or `scripts/check_coverage.ps1 -Table` (Windows, PowerShell) |
| Doc build | `cargo doc --all-features --no-deps --document-private-items` |
| Doc tests | `cargo test --doc --all-features` |

### Important Notes

- **Always run `scripts/quality_check.sh`** once before every `git commit`.
  This is the single source of truth for quality validation — it runs
  formatting, linting, doc checks, and all tests. Use `-v` for verbose
  output if debugging failures. On Unix, you may optionally wrap it with
  `timeout 240` to prevent hangs.
- **Use `cargo nextest run || true` for tests**, never `cargo test` (except
  for doc tests). The `|| true` prevents shell abort due to a known nextest
  issue in this project — **you must still inspect the output and treat any
  test failure as a real failure**.
- **Always run `cargo fmt` and `cargo clippy -- -D warnings`** and fix every
  warning before submitting code.
- Coverage threshold is **75%** line coverage.
- Required tooling: Rust stable, `rustfmt`, `clippy`, `cargo-nextest`.
  For coverage: `cargo-llvm-cov`, plus `jq` and `bc` on Linux/macOS (the
  Windows port `scripts/check_coverage.ps1` parses JSON natively in
  PowerShell and needs neither `jq` nor `bc`).

### CPU-Intensive Operations — Main Agent Only

**NEVER** run the following commands in sub-agents or in parallel:

- `scripts/quality_check.sh` (runs the full test suite + linting + docs)
- `scripts/check_coverage.sh` (runs the full test suite with instrumentation)
- `cargo nextest run` without a `--filter-expr` (runs all 2000+ tests)

These operations are CPU-intensive and will saturate all cores. Running them
in multiple sub-agents simultaneously will cause system overload, timeouts,
and unreliable results.

**Correct workflow:**

- Sub-agents writing tests should run only their own scoped tests using
  `cargo nextest run --filter-expr 'test(module_name)' || true`.
- The **main agent** runs `scripts/quality_check.sh` once after all
  sub-agents have finished and changes are consolidated.
- The main agent runs the quality check **before every `git commit`** to
  ensure all changes pass together.

## Architecture

### Execution Flow

```
main.rs → cli::run() → cli::run_with_config()
  → commands::dispatcher::dispatch_command_with_ref()
    → *_command::execute(args, &dyn ConfigService)
      → core/* and services/*
```

### Layer Overview

```
CLI Layer (src/cli/)          → Argument parsing, user interface
Command Layer (src/commands/) → Business logic per command
Core Layer (src/core/)        → Processing engines, formats, matching
Service Layer (src/services/) → External integrations (AI, audio, VAD)
Config Layer (src/config/)    → DI-based configuration system
```

### Key Design Patterns

- **Dependency injection** — All components receive `&dyn ConfigService` or
  `Arc<dyn ConfigService>`. Never use global state. `ComponentFactory`
  (in `src/core/factory.rs`) centralizes component construction from config.
- **Trait objects for polymorphism** — `SubtitleFormat` (format plugins),
  `AIProvider` (AI backends), `ConfigService` (prod vs test),
  `EnvironmentProvider` (system vs test env).
- **Async throughout** — `tokio` runtime; `async_trait` for trait methods;
  semaphore-limited concurrency.
- **Error handling** — `SubXError` enum via `thiserror` with typed variants,
  exit codes (1–6), and user-friendly messages. Use `crate::Result<T>`
  alias. Propagate with `?`; use `From` impls for automatic conversion.
- **File operations** — `FileManager` provides batch file operations with
  backup support. Rollback covers recorded creations and moves but cannot
  restore removed files.

### Module Guide

| Module | Purpose | Key Types |
|---|---|---|
| `src/cli/` | Argument parsing via clap derive | `Cli`, `Commands`, `*Args`, `InputPathHandler` |
| `src/commands/` | Command implementations | `dispatcher`, `execute()` functions |
| `src/config/` | Configuration with DI | `ConfigService`, `Config`, `TestConfigService`, `TestConfigBuilder` |
| `src/core/factory.rs` | Component wiring | `ComponentFactory` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jim60105/subx-cli](https://github.com/jim60105/subx-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->

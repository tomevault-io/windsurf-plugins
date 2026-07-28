---
trigger: always_on
description: **brush** (Bourne Rusty Shell) is a POSIX- and bash-compatible shell implemented in Rust. It's a multi-crate workspace (~60K lines of Rust code) targeting Linux, macOS, and WSL, with experimental Windows and WASM support. The project emphasizes compatibility testing against bash as an oracle.
---

# GitHub Copilot Coding Agent Instructions for brush

## Project Overview

**brush** (Bourne Rusty Shell) is a POSIX- and bash-compatible shell implemented in Rust. It's a multi-crate workspace (~60K lines of Rust code) targeting Linux, macOS, and WSL, with experimental Windows and WASM support. The project emphasizes compatibility testing against bash as an oracle.

**Key Stats:** Rust 2024 edition, MSRV 1.88.0, 5 main crates, 1500+ compatibility test cases, published to crates.io.

## Critical: Read AGENTS.md First

**BEFORE making any changes, read `/AGENTS.md`.** It contains detailed architecture patterns, testing workflows, and development guidelines specific to this project. The information below supplements (not replaces) AGENTS.md.

## Code Review Checklist

When reviewing PRs, verify:

- [ ] **Documentation**: All exported APIs have rustdoc comments (missing docs = CI failure)
- [ ] **Forbidden patterns**: No `panic`, `unwrap_in_result`, `expect_used`, or `todo` (all denied by clippy)
- [ ] **Error handling**: Uses `thiserror` for crate errors; `anyhow` only in tests
- [ ] **Logging**: Uses `tracing::debug!(target: trace_categories::CATEGORY, "msg")` pattern
- [ ] **Testing**: Compatibility fixes include YAML test cases in `brush-shell/tests/cases/`
- [ ] **Testing**: Builtin changes have tests in `brush-shell/tests/cases/builtin/`
- [ ] **Testing**: Unit tests expected for new public APIs (when feasible) (see AGENTS.md section 2)
- [ ] **Platform code**: Platform-specific code is in `brush-core/src/sys/` modules
- [ ] **Breaking changes**: Public API changes are clearly highlighted and documented
- [ ] **Builder pattern**: Configuration uses builder pattern (see `Shell::builder()`)
- [ ] **Code quality**: Passes `cargo fmt --check` and `cargo clippy` without warnings
- [ ] **Commit format**: Follows [Conventional Commits](https://www.conventionalcommits.org/) (feat:, fix:, docs:, test:)
- [ ] **Dependencies**: No unnecessary cloning (use references when possible)
- [ ] **Cross-platform**: Uses appropriate `cfg(unix)`, `cfg(windows)`, `cfg(target_family = "wasm")`

## Workspace Structure

```
brush/
├── brush-shell/        # CLI application & main entry point
├── brush-interactive/  # Interactive shell (readline, completion)
├── brush-core/         # Core shell runtime & builtins
├── brush-builtins/     # Shell builtin implementations
├── brush-parser/       # AST generation & parsing
├── xtask/             # Build automation tasks
└── docs/              # Diátaxis-structured documentation
```

**Dependency flow:** brush-shell → brush-interactive → brush-core → brush-parser
                                  ↘ brush-builtins ↗

## Build & Validation Commands

### Terminal Command Execution

When running commands that may take more than a few seconds (cargo build, cargo check, cargo test, cargo clippy, cargo xtask, etc.), **run them in background mode and poll for results** rather than blocking. This prevents commands from being cancelled due to timeouts. Use `isBackground: true` with `run_in_terminal`, then use `get_terminal_output` to check results.

### Using xtask (Recommended)

The project provides a `cargo xtask` command that centralizes common development tasks. This is the recommended approach for running checks and tests.

#### Quick Development Cycle

```bash
# Run quick inner-loop checks (~7s warm): fmt, build, lint, unit tests
cargo xtask ci quick

# Run full pre-commit checks (~45s warm): quick + deps, schemas, integration tests
cargo xtask ci pre-commit

# Run with --continue-on-error to see all failures at once
cargo xtask ci pre-commit -k

# Add -v for verbose output showing exact commands being run
cargo xtask -v ci pre-commit
```

#### Individual Checks

```bash
# Format check
cargo xtask check fmt

# Lint check (clippy)
cargo xtask check lint

# Dependency check (cargo-deny)
cargo xtask check deps

# Build check
cargo xtask check build

# Schema check (regenerates and diffs)
cargo xtask check schemas
```

#### Running Tests

```bash
# Run unit tests (fast tests excluding integration binaries)
cargo xtask test unit

# Run integration tests (all workspace tests including compat tests)
cargo xtask test integration

# Run tests with coverage
cargo xtask test integration --coverage --coverage-output codecov.xml
```

### Manual Approach (Alternate)

For finer-grained control or when xtask isn't available:

#### Quick Development Cycle (Use These Frequently)

```bash
# Fast syntax/type checking (< 5 seconds)
cargo check --workspace

# Package-specific checking (even faster)
cargo check --package brush-core

# Format code (ALWAYS run before committing)
cargo fmt --all

# Lint code (ALWAYS run before committing)
cargo clippy --workspace --all-features --all-targets

# Run package-specific tests (fast iteration)
cargo test --package brush-parser
cargo test --package brush-core
```

**Note:** `cargo fmt --check` may show warnings about unstable rustfmt features (`wrap_comments`, `comment_width`) on stable Rust. These are harmless and expected.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reubeno/brush](https://github.com/reubeno/brush) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

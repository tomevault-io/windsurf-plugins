---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Project Overview

Git Workers is an interactive CLI tool for managing Git worktrees, written in Rust. It provides a menu-driven interface for creating, deleting, switching, and renaming worktrees, with shell integration for automatic directory switching.

## Development Commands

### Build and Run

```bash
# Development build
cargo build

# Release build
cargo build --release

# Run directly (development)
cargo run

# Run the binary
./target/debug/gw
./target/release/gw

# Run tests
cargo test

# Run specific test
cargo test test_name

# Run tests single-threaded (for flaky tests)
cargo test -- --test-threads=1

# Run tests with output for debugging
cargo test test_name -- --nocapture

# Run with logging enabled
RUST_LOG=debug cargo run
RUST_LOG=git_workers=trace cargo run
```

### Quality Checks

```bash
# Format check and apply
cargo fmt --check
cargo fmt

# Clippy (linter)
cargo clippy --all-features -- -D warnings

# Type check
cargo check --all-features

# Generate documentation
cargo doc --no-deps --open

# Run all checks (using bun if available)
bun run check

# Coverage report (requires cargo-llvm-cov)
cargo llvm-cov --html --lib --ignore-filename-regex '(tests/|src/main\.rs|src/bin/)' --open
```

### Commit Conventions

- Follow `Conventional Commits` for all commit messages
- Format: `<type>(<scope>)?: <description>`
- Common types in this repository:
  - `feat`: user-facing feature additions
  - `fix`: bug fixes and behavior corrections
  - `refactor`: structural changes without behavior changes
  - `test`: test additions or test-only refactors
  - `docs`: documentation-only changes
  - `chore`: maintenance work with no product behavior impact
  - `ci`: CI or automation workflow changes
  - `build`: build system or dependency management changes
- Keep the subject concise, imperative, and lowercase where natural
- Do not mix structural changes and behavior changes in the same commit
- Examples:
  - `refactor(app): move menu dispatch into app module`
  - `fix(create): preserve selected tag when creating worktree`
  - `test(rename): cover cancel flow in rename prompt`

### Installation

```bash
# Install locally from source
cargo install --path .

# Setup shell integration
./setup.sh

# Or manually add to ~/.bashrc or ~/.zshrc:
source /path/to/git-workers/shell/gw.sh
```

## Current Focus Areas

- Interactive worktree operations are driven from the `app` layer and delegated into `usecases`
- Existing public paths such as `commands`, `infrastructure`, and `repository_info` are kept as compatibility facades
- The project supports shell-assisted directory switching, lifecycle hooks, file copying, tag-based creation, and validated custom paths
- Current refactoring policy prioritizes preserving observable behavior over aggressively removing compatibility layers

## Architecture

### Core Module Structure

```
src/
├── main.rs                 # Thin CLI entry point (`--version` + app startup)
├── lib.rs                  # Public module exports and backward-compatible re-exports
├── app/                    # Menu loop, action dispatch, presenter helpers
├── usecases/               # Main worktree operations (create/delete/list/rename/switch/search)
├── adapters/               # Config, shell, filesystem, Git, UI, and hook adapters
├── domain/                 # Repository context and domain-level helpers
├── commands/               # Backward-compatible facades over usecases
├── config.rs               # Configuration model and access helpers
├── repository_info.rs      # Backward-compatible facade for repo context display
├── infrastructure/         # Backward-compatible exports for older module paths
├── core/                   # Legacy core logic retained during migration
├── ui.rs                   # User interface abstraction used by prompts and tests
├── input_esc_raw.rs        # ESC-aware input helpers
├── constants.rs            # Centralized strings and formatting constants
├── support/                # Terminal and styling support utilities
└── utils.rs                # Shared utilities and compatibility helpers
```

### Dependency Direction

- `main` -> `app`
- `app` -> `usecases`
- `usecases` -> `adapters`, `domain`, `ui`, `config`, `infrastructure`
- `commands` and `repository_info` should stay thin and delegate to the newer modules
- Public compatibility paths are intentionally preserved unless a breaking change is explicitly planned

### Technology Stack

- **dialoguer + console**: Interactive CLI (Select, Confirm, Input prompts)
- **git2**: Git repository operations (branch listing, commit info)
- **std::process::Command**: Git CLI invocation (worktree add/prune)
- **colored**: Terminal output coloring
- **fuzzy-matcher**: Worktree search functionality
- **indicatif**: Progress bar display

### Shell Integration System

Automatic directory switching on worktree change requires special implementation due to Unix process restrictions:

1. Binary writes path to file specified by `GW_SWITCH_FILE` env var
2. Shell function (`shell/gw.sh`) reads the file and executes `cd`
3. Legacy fallback: `SWITCH_TO:/path` marker on stdout

### Hook System Design


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wasabeef/git-workers](https://github.com/wasabeef/git-workers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->

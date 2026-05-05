---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

```bash
cargo build                                    # Debug build
cargo build --release                          # Release build (<15MB target)
cargo test --lib --workspace                   # Run all unit tests
cargo test --lib -p agent-desktop-core         # Test core crate only
cargo test --lib -p agent-desktop-macos        # Test macOS crate only
cargo test test_name                           # Run a single test by name
cargo clippy --all-targets -- -D warnings      # Lint (must pass, zero warnings)
cargo fmt --all -- --check                     # Format check
cargo fmt --all                                # Auto-format
cargo tree -p agent-desktop-core               # Verify no platform crate leaks (CI enforces)
```

Run the binary: `./target/release/agent-desktop snapshot --app Finder -i`

## Pre-commit Hook

The repo ships a pre-commit hook at `.githooks/pre-commit` that runs `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test --lib --workspace` against staged Rust changes. Wire it up once after cloning:

```bash
git config core.hooksPath .githooks
```

Bypass for an emergency commit with `git commit --no-verify` or `SKIP_PRECOMMIT=1 git commit ...`.

## Project Overview

Cross-platform Rust CLI + MCP server enabling AI agents to observe and control desktop applications via native OS accessibility trees.

## Git & Commits

- All commits are authored by **Lahfir**
- NEVER add `Co-Authored-By` lines, AI attribution badges, or "Generated with" footers
- NEVER include co-committers of any kind
- **Conventional Commits required.** Every commit message must use a type prefix:
  - `feat:` — new feature (triggers minor version bump)
  - `fix:` — bug fix (triggers patch version bump)
  - `feat!:` or `BREAKING CHANGE:` footer — breaking change (triggers major version bump)
  - `docs:` — documentation only
  - `style:` — formatting, no code change
  - `refactor:` — code change that neither fixes a bug nor adds a feature
  - `chore:` — maintenance tasks, dependencies
  - `ci:` — CI/CD changes
  - `test:` — adding or fixing tests
- Format: `type: concise imperative description` (lowercase type, no capital after colon)
- Focus on "why" not "what"
- Examples: `feat: add scroll-to command`, `fix: prevent stale ref on window resize`, `ci: add binary size check`

## Core Principle

agent-desktop is NOT an AI agent. It is a tool that AI agents invoke. It outputs structured JSON with ref-based element identifiers. The observation-action loop lives in the calling agent.

## Architecture

### Workspace Layout

```
agent-desktop/
├── Cargo.toml              # workspace: members, shared deps
├── rust-toolchain.toml     # pinned Rust version
├── clippy.toml             # project-wide lint config
├── crates/
│   ├── core/               # agent-desktop-core (platform-agnostic)
│   │   └── src/
│   │       ├── ref_alloc.rs      # Shared ref helpers (INTERACTIVE_ROLES, is_collapsible)
│   │       ├── snapshot_ref.rs   # Ref-rooted drill-down (run_from_ref)
│   │       └── commands/         # one file per command
│   ├── macos/              # agent-desktop-macos (Phase 1)
│   ├── windows/            # agent-desktop-windows (stub → Phase 2)
│   ├── linux/              # agent-desktop-linux (stub → Phase 2)
│   └── ffi/                # agent-desktop-ffi (cdylib + cbindgen C ABI)
├── src/                    # agent-desktop binary (entry point)
│   ├── main.rs             # entry point, permission check, JSON envelope
│   ├── cli.rs              # clap derive enum (Commands)
│   ├── cli_args.rs         # all command argument structs
│   ├── dispatch.rs         # command dispatcher + parse helpers
│   └── batch_dispatch.rs   # batch command execution
├── docs/
│   └── solutions/          # documented solutions to past problems (bugs, best practices, workflow patterns), organized by category with YAML frontmatter (module, tags, problem_type); relevant when implementing or debugging in documented areas
└── tests/
    ├── fixtures/           # golden JSON snapshots
    └── integration/        # macOS CI integration tests
```

### Dependency Inversion (Non-Negotiable)

- `agent-desktop-core` defines the `PlatformAdapter` trait and all shared types
- Platform crates (`macos`, `windows`, `linux`) implement the trait
- **Core NEVER imports platform crates.** Platform crates NEVER import each other.
- Two legitimate wiring points bring platform → core together:
  1. The binary crate (`src/`) — CLI consumers
  2. The FFI crate (`crates/ffi/`) — cdylib consumers (Python, Swift, Go, Node, C++)
- CI enforces core isolation: `cargo tree -p agent-desktop-core` must contain zero platform crate names

### Platform Selection

Compile-time via `#[cfg(target_os)]` in `build_adapter()`. Agents never specify platform — `agent-desktop snapshot -i` works identically on macOS, Windows, and Linux.

```rust
fn build_adapter() -> impl PlatformAdapter {
    #[cfg(target_os = "macos")]
    { agent_desktop_macos::MacOSAdapter::new() }

    #[cfg(target_os = "windows")]
    { agent_desktop_windows::WindowsAdapter::new() }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lahfir/agent-desktop](https://github.com/lahfir/agent-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

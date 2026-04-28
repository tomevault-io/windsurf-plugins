---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Principles

**Target User:** Power users and agentic-forward engineers who want speed, control, and isolation. Users who run multiple AI agents simultaneously and need clean environment separation.

**Single-Developer Tool:** No multi-tenant complexity. Optimize for the solo developer managing parallel AI workflows.

**KISS:** Keep it simple and easily understandable over complex and "clever". First principles thinking.

**YAGNI:** Justify every line of code against the problem it solves. Is it needed?

**Type Safety (CRITICAL):** Rust's type system is a feature, not an obstacle. Use it fully.

**No Silent Failures:** This is a developer tool. Developers need to know when something fails. Never swallow errors, never hide failures behind fallbacks without logging, never leave things "behind the curtain". If config is wrong, say so. If an operation fails, surface it. Explicit failure is better than silent misbehavior.

## Git as First-Class Citizen

KILD is built around git worktrees. Let git handle what git does best:

- **Surface git errors to users** for actionable issues (conflicts, uncommitted changes, branch already exists)
- **Handle expected failures gracefully** (missing directories during cleanup, worktree already removed)
- **Trust git's natural guardrails** (e.g., git2 refuses to remove worktree with uncommitted changes - surface this, don't bypass it)
- **Branch naming:** KILD creates `kild/<branch>` branches automatically for isolation using git-native namespacing. The worktree admin name (`kild-<branch>`) is filesystem-safe and decoupled from the branch name via `WorktreeAddOptions::reference()`

## Code Quality Standards

All PRs must pass before merge:

```bash
cargo fmt --check              # Formatting (0 violations)
cargo clippy --all -- -D warnings  # Linting (0 warnings, enforced via -D)
cargo test --all               # All tests pass
cargo build --all              # Clean build
```

**Tooling:**

- `cargo fmt` - Rustfmt with default settings
- `cargo clippy` - Strict linting, warnings treated as errors
- `thiserror` - For error type definitions
- `tracing` - For structured logging (JSON output)

## Build & Development Commands

```bash
# Build
cargo build --all              # Build all crates
cargo build -p kild-core       # Build specific crate
cargo build -p kild-paths      # Build paths crate
cargo build -p kild-protocol   # Build protocol types crate

# Test
cargo test --all               # Run all tests
cargo test -p kild-core        # Test specific crate
cargo test -p kild-paths       # Test paths crate
cargo test -p kild-protocol    # Test protocol types crate
cargo test test_name           # Run single test by name

# Lint & Format
cargo fmt                      # Format code
cargo fmt --check              # Check formatting
cargo clippy --all -- -D warnings  # Lint with warnings as errors

# Run
cargo run -p kild -- create my-branch --agent claude
cargo run -p kild -- create my-branch --agent claude --note "Working on auth feature"
cargo run -p kild -- create my-branch --no-agent       # Open bare terminal with $SHELL
cargo run -p kild -- create my-branch --daemon         # Launch in daemon-owned PTY
cargo run -p kild -- create my-branch --no-daemon      # Force external terminal (override config)
cargo run -p kild -- list
cargo run -p kild -- list --json                 # JSON object with sessions array and fleet_summary
cargo run -p kild -- status my-branch --json     # JSON output for single kild
cargo run -p kild -- -v list                     # Verbose mode (enable JSON logs)
cargo run -p kild -- cd my-branch                # Print worktree path for shell integration
cargo run -p kild -- open my-branch              # Open new agent in existing kild (auto-detects runtime mode from session)
cargo run -p kild -- open my-branch --agent kiro # Open with different agent
cargo run -p kild -- open my-branch --no-agent   # Open bare terminal with $SHELL (no agent)
cargo run -p kild -- open my-branch --resume     # Resume previous agent session (restore conversation context)
cargo run -p kild -- open my-branch -r           # Short form of --resume
cargo run -p kild -- open my-branch --daemon     # Override: force daemon-owned PTY
cargo run -p kild -- open my-branch --no-daemon  # Override: force external terminal
cargo run -p kild -- open --all                  # Open agents in all stopped kilds
cargo run -p kild -- open --all --agent claude   # Open all stopped kilds with specific agent
cargo run -p kild -- open --all --no-agent       # Open bare terminals in all stopped kilds
cargo run -p kild -- open --all --resume         # Resume all stopped kilds with previous session context
cargo run -p kild -- code my-branch              # Open worktree in editor (config > $EDITOR > code)
cargo run -p kild -- code my-branch --editor vim # Override editor
cargo run -p kild -- focus my-branch             # Bring terminal window to foreground
cargo run -p kild -- hide my-branch              # Minimize/hide terminal window
cargo run -p kild -- hide --all                  # Hide all active kild windows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wirasm/kild](https://github.com/Wirasm/kild) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Project Overview

**vibes** — The vibe engineering mech suit.

vibes augments *you*—the human developer—with AI-powered superpowers: remote session control, persistent context, and a learning system that remembers what works. You stay in command; vibes amplifies your reach.

### Architecture

| Crate | Purpose |
|-------|---------|
| **vibes-core** | Shared library (sessions, events, plugins, auth, tunnel) |
| **vibes-server** | HTTP/WebSocket server (axum-based) |
| **vibes-cli** | CLI binary, connects to daemon via WebSocket |
| **vibes-iggy** | EventLog backed by Apache Iggy |
| **vibes-plugin-api** | Published crate for plugin authors |
| **vibes-introspection** | Harness detection and capability discovery |
| **vibes-groove** | Continual learning plugin (under `plugins/`) |
| **web-ui** | TanStack frontend embedded via rust-embed |

See [docs/VISION.md](docs/VISION.md) for product vision and [docs/board/README.md](docs/board/README.md) for project status.

### Event Sourcing (CRITICAL)

**vibes is a fully event-sourced system.** All state derives from events stored in Apache Iggy.

| Principle | Description |
|-----------|-------------|
| **Events are the source of truth** | State is derived from replaying events, never stored directly |
| **Iggy is the event store** | All domain events go through vibes-iggy |
| **Projections for queries** | SQLite/other stores are read-optimized projections, rebuilt from events |

**Before implementing any storage:**
1. Define the events that capture state changes
2. Store events in Iggy
3. Build projections as needed for query performance

**ASK before making architectural decisions** like:
- Choosing to store state directly instead of as events
- Adding a new database or storage mechanism
- Changing how events flow through the system
- Deviating from the event-sourced pattern

## Setup

**Nix flake** with **direnv** for reproducible environments:

```bash
cd vibes                                    # direnv auto-loads Nix shell
direnv allow                                # First time only
just setup-hooks                            # Enable git hooks (pre-commit + post-checkout)
just build                                  # Build vibes + iggy-server
```

Submodules are initialized automatically by the `post-checkout` hook when creating worktrees or cloning.

### Shared Build Cache

All worktrees share build caches for faster builds:

| Cache | Location | Purpose |
|-------|----------|---------|
| Rust artifacts | `~/.cache/cargo-target/vibes/` | Cargo compilation cache |
| Turbo cache | `~/.cache/turbo/vibes/` | npm workspace build cache |

This means:
- Fresh worktrees reuse compiled artifacts from other worktrees
- Incremental compilation is shared across worktrees
- Web-ui `dist/` is restored from turbo cache on checkout (~80ms)

**Binary Isolation:** `just build` copies final binaries (`vibes`, `iggy-server`) to `./target/debug/` in each worktree. Tests automatically find these worktree-local binaries, preventing cross-worktree binary clobbering.

**WARNING:** `cargo clean` will delete Rust artifacts for ALL worktrees. Use with caution.

## Commands

**Always use `just` over raw cargo commands.**

### Top-Level Commands

| Command | Purpose |
|---------|---------|
| `just` | List all available commands |
| `just setup` | Full setup for new developers |
| `just build` | Debug build (vibes + iggy-server) |
| `just pre-commit` | All checks before committing |

### Module Commands

Commands are organized into modules. Use `just <module>` to see available subcommands.

| Module | Commands | Examples |
|--------|----------|----------|
| `just tests` | `run`, `all`, `integration`, `watch`, `one <name>` | `just tests run` |
| `just quality` | `check`, `clippy`, `fmt`, `fmt-check`, `mutants` | `just quality clippy` |
| `just coverage` | `report`, `html`, `summary`, `lcov`, `package <pkg>` | `just coverage summary` |
| `just builds` | `debug`, `release`, `dev` | `just builds dev` |
| `just web` | `build`, `typecheck`, `test`, `install`, `e2e`, `e2e-setup` | `just web build` |
| `just plugin` | `list`, `install-groove`, `uninstall-groove` | `just plugin list` |
| `just board` | `status`, `generate`, `new`, `start`, `done`, `link` | `just board status` |

### Board Commands

| Command | Purpose |
|---------|---------|
| `just board` | Show available commands |
| `just board generate` | Regenerate board README.md |
| `just board status` | Show board status |
| `just board new story "title"` | Create new story |
| `just board new epic "name"` | Create new epic |
| `just board new milestone "name"` | Create new milestone |
| `just board start <id>` | Move story to in-progress |
| `just board done <id>` | Move story to done |
| `just board ice <id>` | Move story to icebox (blocked/deferred) |
| `just board thaw <id>` | Move story from icebox to backlog |
| `just board start-milestone <id>` | Set milestone to in-progress |
| `just board done-milestone <id>` | Set milestone to done |
| `just board done-epic <id>` | Set epic to done |

### Verification Commands

Visual verification captures screenshots and videos to document system behavior.

| Command | Purpose |
|---------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [run-vibes/vibes](https://github.com/run-vibes/vibes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->

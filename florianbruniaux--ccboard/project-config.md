---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ccboard** is a unified dashboard for Claude Code management, providing both TUI and web interfaces from a single binary to visualize sessions, stats, configuration, hooks, agents, costs, and history from `~/.claude` directories.

**Stack**: Rust workspace with 4 crates, Ratatui (13-tab TUI), Axum (Web API backend), Leptos (WASM frontend), Arc + parking_lot for concurrency.

## Workspace Architecture

This is a Cargo workspace with a layered architecture:

```
ccboard/                     # Root binary - CLI entry point
├─ ccboard-core/             # Shared data layer (parsers, models, store, watcher)
├─ ccboard-tui/              # Ratatui frontend (13 tabs)
└─ ccboard-web/              # Axum API backend + Leptos WASM frontend
```

**Dependency flow**: `ccboard` → `ccboard-tui` + `ccboard-web` → `ccboard-core`

**Core principle**: Single binary, dual frontends sharing a thread-safe `DataStore`.

## Development Commands

### Build & Run

```bash
# Build all crates
cargo build --all

# Run TUI (default mode)
cargo run

# Run web interface
cargo run -- web --port 3333

# Run both TUI and web
cargo run -- both --port 3333

# Print stats and exit
cargo run -- stats

# Session management commands
cargo run -- search "query" --limit 10          # Search sessions
cargo run -- search "bug" --since 7d            # Search last 7 days
cargo run -- recent 10                          # Show 10 most recent sessions
cargo run -- recent 5 --json                    # JSON output
cargo run -- info <session-id>                  # Show session details
cargo run -- resume <session-id>                # Resume session in Claude CLI
cargo run -- clear-cache                        # Clear SQLite cache

# Specify Claude home directory
cargo run -- --claude-home ~/.claude --project /path/to/project

# Frontend (Leptos/WASM)
trunk serve --port 3333                             # Serve frontend on http://127.0.0.1:3333
```

**Full stack workflow**:
```bash
# Terminal 1: Backend API
cargo run -- web --port 8080

# Terminal 2: Frontend
trunk serve --port 3333
# Frontend communicates with backend via http://localhost:8080/api/*
```

### Testing

```bash
# Run all tests
cargo test --all

# Run tests for specific crate
cargo test -p ccboard-core

# Run tests with logging
RUST_LOG=debug cargo test

# Run integration tests (requires real ~/.claude data)
cargo test --all-features
```

### Quality Checks

```bash
# Format code (REQUIRED before commit)
cargo fmt --all

# Clippy (MUST pass with zero warnings)
cargo clippy --all-targets

# Pre-commit checklist
cargo fmt --all && cargo clippy --all-targets && cargo test --all
```

### Development Workflow

```bash
# Watch and rebuild TUI
cargo watch -x 'run'

# Watch and rebuild web
cargo watch -x 'run -- web'

# Run with debug logging
RUST_LOG=ccboard=debug cargo run
```

## Core Architecture Patterns

### DataStore: Central Thread-Safe State

`ccboard-core/src/store.rs` is the single source of truth, shared by both TUI and web frontends:

- **Arc<DataStore>** for shared ownership (replaced DashMap in v0.4.0, ~50x memory reduction)
- **parking_lot::RwLock** for stats/settings (low contention, frequent reads, better fairness than std)
- **SQLite Cache** for session metadata (89x faster than JSONL parsing, on-demand full content loading)
- **EventBus** (tokio broadcast) for live updates across frontends

**Key methods**:
- `initial_load()` → Returns `LoadReport` for graceful degradation
- `reload_stats()` → Called by file watcher
- `update_session(path)` → Called when session file changes
- `stats()`, `settings()`, `sessions_by_project()` → Read accessors

### Graceful Degradation Pattern

All parsers return `Option<T>` and populate `LoadReport` instead of failing fast:

```rust
pub struct LoadReport {
    pub stats_loaded: bool,
    pub settings_loaded: bool,
    pub sessions_scanned: usize,
    pub sessions_failed: usize,
    pub errors: Vec<LoadError>,
}
```

**Rationale**: ccboard should display partial data if some files are corrupted/missing. Only fatal errors prevent UI launch.

### Parsing Strategy

Located in `ccboard-core/src/parsers/`:

- **stats.rs** → `stats-cache.json` (serde_json direct, retry logic for file contention)
- **settings.rs** → JSON merge (global → project → local priority)
- **session_index.rs** → JSONL streaming (BufReader line-by-line, skip malformed)
  - **Lazy metadata extraction**: Only parse first + last line to extract timestamps, message count, models used
  - **Full parse on demand**: Session content loaded when user requests detail view
- **Frontmatter** (agents/commands/skills) → Custom YAML split + serde_yaml

**Performance constraint**: With 1000+ sessions and 2.5GB of JSONL data, full parse at startup is unacceptable. Metadata-only scan targets <2s.

### Concurrency Model

- **Initial scan**: `tokio::spawn` per project directory (up to 8 concurrent)
- **File watcher**: `notify` crate with 500ms debounce (notify-debouncer-mini)
- **EventBus**: `tokio::sync::broadcast` with 256 capacity
- **SQLite Cache**: Thread-safe reads, lazy on-demand content loading


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlorianBruniaux/ccboard](https://github.com/FlorianBruniaux/ccboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->

---
trigger: always_on
description: Rust CLI tool that watches your coding agent sessions, discovers what you keep repeating, and turns those patterns into persistent context — automatically. Session over session, your agent gets better without you maintaining its context by hand.
---

# Retro — Active Context Curator for AI Coding Agents

Rust CLI tool that watches your coding agent sessions, discovers what you keep repeating, and turns those patterns into persistent context — automatically. Session over session, your agent gets better without you maintaining its context by hand.

## Architecture

### v2 "The Watcher" (primary)

Five-layer pipeline, data flows upward:

```
Surfaces        (TUI dashboard, in-session briefing, CLI)
Projectors      (Claude Code — pluggable for other agents)
Knowledge Store (graph-modeled in SQLite: nodes + edges)
Analyzers       (pattern discovery, scope classification)
Observers       (session watcher via mtime polling)
Scheduled Runner (launchd periodic job)
```

`retro run` executes the full pipeline. `retro start` installs a launchd job that runs it every 5 minutes. `retro dash` opens the TUI dashboard for reviewing suggestions and browsing knowledge.

### v1 (legacy, still functional)

Three-stage pipeline: **Ingestion** → **Analysis** → **Projection**. Driven by post-commit hooks (`--auto` flag, now deprecated). All v1 commands still work.

### Storage

`~/.retro/` contains: SQLite DB (WAL mode, schema v4), JSONL audit log, config.toml, `briefings/` directory, `runner.log`.

## Repo Structure

Cargo workspace with two crates:
- `crates/retro-core/` — library crate (all logic, models, DB, analysis, knowledge graph, observers, runner helpers)
- `crates/retro-cli/` — binary crate (clap commands, TUI dashboard, launchd integration)
  - `src/tui/` — TUI module (app state, rendering, event handling)
  - `src/launchd.rs` — macOS launchd plist generation and management
- `tests/` — fixtures and integration tests
- `scenarios/` — scenario-based integration tests (see [scenarios/README.md](scenarios/README.md))

## Build & Test

```bash
# Build (requires Rust toolchain and C compiler for bundled SQLite)
cargo build

# Run unit tests
cargo test

# Run scenario tests
./scenarios/README.md  # see file for test runner usage

# Always run tests before committing
cargo test && cargo run -- --help  # verify build

# Clean install testing
retro init --uninstall --purge && cargo build --release && ./target/release/retro init
```

## Commands Overview

### v2 Commands (new)

| Command | Purpose |
|---------|---------|
| `retro run [--dry-run]` | Run the full v2 pipeline (observe → ingest → analyze → project) |
| `retro start` | Start the scheduled runner (launchd on macOS) |
| `retro stop` | Stop the scheduled runner |
| `retro dash` | Open the TUI dashboard (review suggestions, browse knowledge) |

### Core Commands

| Command | Purpose |
|---------|---------|
| `retro init` | Initialize retro (creates DB, config, launchd job, briefing skill) |
| `retro ingest [--auto]` | Scan Claude Code session files and save to DB |
| `retro analyze [--dry-run] [--auto]` | AI-powered pattern discovery from sessions |
| `retro patterns` | List discovered patterns |
| `retro apply [--dry-run] [--auto] [--global]` | Generate skills/CLAUDE.md from patterns (saved as PendingReview) |
| `retro review` | Review and approve/skip/dismiss pending projections |
| `retro sync` | Sync PR state, reset patterns from closed PRs |
| `retro curate [--dry-run]` | AI-assisted CLAUDE.md editing (direct file write) |
| `retro diff [--global]` | Preview changes to CLAUDE.md or global agents |
| `retro status` | Show summary of sessions, patterns, projections |
| `retro clean [--dry-run]` | Archive stale patterns |
| `retro audit [--dry-run]` | Context audit (detects inconsistencies) |
| `retro log [--since <days>]` | View audit log entries |
| `retro hooks remove` | Remove git hooks |
| `retro init --uninstall [--purge]` | Uninstall retro (removes launchd plist, hooks, optionally data) |

Note: `--auto` flag is deprecated in v2. Use `retro start` for automatic background operation.

## Key Design Decisions

### Core Architecture

- **Rust, sync only** — no tokio, no async. `std::process::Command` for spawning `claude` CLI and `git`/`gh`.
- **No git2 crate** — shell out to `git` and `gh` directly for simplicity and reliability.
- **SQLite bundled** — `rusqlite` with `bundled` feature. WAL mode always. Schema versioned via `PRAGMA user_version`.
- **Error handling** — `thiserror` in retro-core, `anyhow` in retro-cli. `CoreError` implements `std::error::Error` — use `?` directly in CLI commands.

### AI Backend

- **Sync trait** — `AnalysisBackend` trait with `json_schema: Option<&str>` parameter.
- **Primary impl** — `ClaudeCliBackend` uses `claude -p - --output-format json` (prompt piped via stdin to avoid ARG_MAX issues).
- **Structured output** — JSON-producing calls pass `--json-schema` for constrained decoding (guaranteed valid JSON, no sanitization needed). Schema constants: `ANALYSIS_RESPONSE_SCHEMA` (analysis/mod.rs), `SKILL_VALIDATION_SCHEMA` (projection/skill.rs), `AUDIT_RESPONSE_SCHEMA` (curator.rs), `GRAPH_ANALYSIS_RESPONSE_SCHEMA` (analysis/mod.rs).
- **CLI quirks**:
  - `--json-schema` conflicts with `--tools ""` on large prompts — only pass `--tools ""` when NOT using `--json-schema`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ImanHashemi/retro](https://github.com/ImanHashemi/retro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

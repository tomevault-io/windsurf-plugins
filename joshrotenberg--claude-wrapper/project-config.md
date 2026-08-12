---
trigger: always_on
description: Guidance for AI assistants working on this repo.
---

# AGENTS.md

Guidance for AI assistants working on this repo.

## What this is

`claude-wrapper` is a type-safe Rust wrapper around the Claude Code CLI. Each subcommand is a builder that produces a typed result. Execution is via tokio (default) or `std::thread` (with the `sync` feature). Built in: long-lived `DuplexSession` for hosts (one child held open across turns; mid-turn interrupts, permission handlers, broadcast subscribers), transient `Session` for short-lived processes (subprocess-per-turn with `--resume`, cumulative cost + history, optional `BudgetTracker` hard-stops, streaming), typed tool-permission patterns (`ToolPattern`), retry policy, and an `McpConfigBuilder` for programmatic `.mcp.json` generation. It also exposes a read-side introspection layer that parses Claude Code's on-disk state (sessions, agents, skills, custom commands, settings, background jobs, worktrees) directly, without spawning the CLI.

The repo is a single library crate. Frontends built on it (CLIs, MCP servers) live in
their own repos; see [`cr`](https://github.com/joshrotenberg/cr).

## Layout

Flat layout:

```
Cargo.toml        # claude-wrapper package + deps + feature flags
src/              # library
tests/            # integration tests + fake-claude.sh
examples/         # runnable examples (async-only, single-file)
```

## Build and test

Run the full pre-commit checklist before every commit:

```bash
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --lib --all-features
cargo test --doc --all-features
```

The crate has overlapping feature gates. CI exercises the interesting
combinations; if you add a new code path behind a feature, verify:

```bash
# Default: ["async", "json", "tempfile"] -- the happy path
cargo build
cargo clippy --all-targets -- -D warnings

# All features enabled (adds `sync`)
cargo build --all-features

# Sync-only, no tokio in the runtime dep tree
cargo build --no-default-features --features "json,sync"
cargo clippy --all-targets --no-default-features --features "json,sync" -- -D warnings
```

Integration tests against the bundled `fake-claude.sh`:

```bash
# Async paths
cargo test --test fake_claude --all-features

# Sync paths (exec, retry, streaming, command surface)
cargo test --test fake_claude_sync --all-features
cargo test --test fake_claude_sync --no-default-features --features "json,sync"
```

Integration tests against a real `claude` binary (ignored by default, requires auth):

```bash
cargo test --test integration -- --ignored
```

## Architecture

`claude-wrapper` is a two-layer builder:

1. **`Claude` client** -- shared config (binary path, working dir, env, timeout, global args, default retry policy). Built via `Claude::builder()`.
2. **Command builders** -- per-subcommand options. Each implements the `ClaudeCommand` trait. Async callers use `cmd.execute(&claude).await`; sync callers enable the `sync` feature, `use claude_wrapper::ClaudeCommandSyncExt`, and call `cmd.execute_sync(&claude)`.

Key modules:

- `src/lib.rs` -- `Claude` + `ClaudeBuilder`, top-level re-exports, `cli_version` / `cli_version_sync`
- `src/command/mod.rs` -- `ClaudeCommand` trait (async `execute` gated on `feature = "async"`), `ClaudeCommandSyncExt` blanket-impl trait (sync)
- `src/command/query.rs` -- `QueryCommand`, the workhorse; has both `execute` / `execute_json` and `execute_sync` / `execute_json_sync`
- `src/command/mcp.rs` -- MCP server management
- `src/command/` also has one module per remaining subcommand: `agents.rs` (deprecated), `auth.rs`, `auto_mode.rs`, `doctor.rs`, `install.rs`, `marketplace.rs`, `plugin.rs`, `project.rs`, `raw.rs`, `update.rs`, `version.rs`
- `src/exec.rs` -- process spawning (tokio for async, `std::process` + `wait-timeout` for sync), concurrent pipe drain, timeout cleanup
- `src/streaming.rs` -- `stream_query` / `stream_query_sync` for NDJSON streaming
- `src/session.rs` -- `Session` for multi-turn conversations from short-lived processes (async-only; subprocess per turn; holds `Arc<Claude>`, auto-threads session_id, tracks history/cost, supports streaming, optional `BudgetTracker` attachment)
- `src/duplex.rs` -- `DuplexSession` for long-running hosts (async + json; one `claude` subprocess held open in stream-json mode across many turns; supports `subscribe`, `interrupt`, and an async `PermissionHandler` for mid-turn permission decisions)
- `src/conversation.rs` -- `Conversation`, host-side bookkeeping over `DuplexSession` (rolling `TurnResult` history, cumulative cost, optional `BudgetTracker` hard-stop). Gated on `json` + `async`
- `src/budget.rs` -- `BudgetTracker` with fire-once warn/exceeded callbacks and `Error::BudgetExceeded` hard-stop
- `src/tool_pattern.rs` -- `ToolPattern` for typed `--allowed-tools` / `--disallowed-tools` patterns (`tool`, `tool_with_args`, `all`, `mcp` constructors; `parse()` validation; loose `From<&str>` for back-compat)
- `src/slash.rs` -- builders for slash-command strings (e.g. `/compact`) to feed stream-json input
- `src/mcp_config.rs` -- `McpConfigBuilder` for generating `.mcp.json` files
- `src/retry.rs` -- `RetryPolicy`, `BackoffStrategy`, `with_retry` (async) and `with_retry_sync`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshrotenberg/claude-wrapper](https://github.com/joshrotenberg/claude-wrapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

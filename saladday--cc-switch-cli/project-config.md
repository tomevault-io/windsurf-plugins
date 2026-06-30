---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

The Rust crate lives in `src-tauri/`. Run Cargo commands from that directory unless a command explicitly targets repository-root assets or scripts.

```bash
cd src-tauri

cargo run                                  # Run cc-switch in interactive mode
cargo run -- provider list                 # Run a specific CLI command
cargo run -- --app codex provider list     # Run a command for a specific app
cargo run -- proxy show                    # Inspect proxy state
cargo run -- env tools                     # Check local CLI tools
cargo build --release                      # Build release binary at target/release/cc-switch

cargo fmt                                  # Format Rust code
cargo fmt --check                          # Check formatting, matching CI
cargo clippy                               # Run lints
cargo test                                 # Run all tests
cargo test provider_switch                 # Run tests whose names contain provider_switch
cargo test --test provider_commands        # Run a single integration test target
cargo test --features test-hooks           # Run tests with the test-hooks feature enabled
```

The repository pins Rust through `src-tauri/rust-toolchain.toml` to Rust 1.91.1 with `rustfmt` and `clippy`. CI currently runs `cargo fmt --check` for changes under `src-tauri/**`.

## Project overview

CC-Switch CLI is a Rust command-line management tool for Claude Code, Codex, Gemini, OpenCode, Hermes, and OpenClaw. It manages provider configurations, MCP servers, prompts, skills, WebDAV sync, local proxy routes, failover, daemon/start flows, deep-link imports, workspace memory files, and environment checks.

The main crate is `src-tauri/`; the repository root contains docs, assets, install/update scripts, packaging metadata, and Nix files.

Key Rust entry points:

- `src/main.rs` parses CLI arguments, initializes logging, creates startup state for most commands, and dispatches to command handlers.
- `src/lib.rs` declares crate modules and re-exports public types used by integration tests and command code.
- `src/cli/mod.rs` defines the top-level Clap CLI, global `--app` flag, and command enum.
- `src/cli/commands/` contains direct command implementations for providers, MCP, prompts, skills, config/WebDAV, proxy, failover, Hermes, start, daemon, env, update, completions, and internal commands.
- `src/commands/` contains library command helpers that are not top-level Clap subcommands, including OpenClaw workspace file and daily memory operations.
- `src/cli/interactive/` and `src/cli/tui/` contain the interactive ratatui UI, runtime action handlers, forms, overlays, route state, and UI rendering.
- `src/services/` contains durable business logic used by commands and the TUI: providers, auth, MCP, prompts, skills, proxy, WebDAV sync, stream checks, speed tests, environment checks, visible apps, subscription/coding-plan quota checks, and state coordination.
- `src/database/` is the SQLite persistence layer. `Database` owns a mutex-wrapped rusqlite connection, schema creation/migration, backups, and DAO modules for providers, MCP, prompts, skills, settings, proxy state, stream checks, universal providers, and failover queues.
- `src/app_config.rs`, `src/provider.rs`, and app-specific config modules (`claude_*`, `codex_config.rs`, `gemini_*`, `hermes_config.rs`, `opencode_config.rs`, `openclaw_config.rs`) define the shared configuration model and live-file adapters for supported apps.
- `src/deeplink/` implements the `ccswitch://v1/import?...` import protocol for provider resources and is exported through `lib.rs` for tests and callers.
- `src/proxy/` implements the local multi-app proxy with Axum handlers, request forwarding, provider routing/failover, provider-specific transformations, response/stream handling, usage logging, model mapping, cache/thinking rectifiers, circuit breaking, and metrics.
- `src/daemon/` implements the Unix supervisor daemon, IPC protocol, logging, pidfile, and restart support.
- `src/store.rs` defines `AppState`, which ties together the database, an in-memory `MultiAppConfig` snapshot, startup live-config imports/recovery, and `ProxyService`.

## State and configuration model

CC-Switch stores core state in SQLite at `~/.cc-switch/cc-switch.db` by default, or under `$CC_SWITCH_CONFIG_DIR/cc-switch.db` when `CC_SWITCH_CONFIG_DIR` is set. `~/.cc-switch/settings.json` stores app settings, `~/.cc-switch/skills/` stores installed skill source files, and `~/.cc-switch/backups/` holds rotating backups.

Legacy `config.json` and `skills.json` are migration/import sources only. `AppState::try_new()` validates and migrates legacy files into SQLite when needed, exports database state into a `MultiAppConfig` snapshot, seeds defaults, migrates old common-config semantics, and constructs `ProxyService`. `AppState::try_new_with_startup_recovery()` also imports live provider configs and recovers proxy takeovers when needed. `AppState::save()` persists the in-memory snapshot back to SQLite.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaladDay/cc-switch-cli](https://github.com/SaladDay/cc-switch-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

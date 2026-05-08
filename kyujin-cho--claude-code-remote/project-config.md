---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Claude Code hooks integration with messaging platforms (Telegram, Discord, Signal) written in Rust (~4MB Telegram-only, ~8MB with Discord, ~30MB with Signal).

Features:
- Intercept Claude Code permission requests via hooks
- Send notifications to users via Telegram (inline keyboards), Discord (buttons), or Signal (text-based)
- Receive user decisions (approve/deny/always allow) through messaging platforms
- Respond back to Claude Code with the user's decision
- Job completion notifications via Stop hooks
- Generic event handler for all Claude Code hook events (SessionStart, PostToolUseFailure, TaskCompleted, etc.)
- Configurable event filters to control notification volume
- Discord support via optional `--features discord` build flag (MIT/Apache 2.0)
- Signal support via optional `--features signal` build flag (AGPL-3.0 licensed)

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Claude Code    │────▶│  Hook Handler    │────▶│  Telegram Bot   │
│  (Permission    │     │  (Rust)          │     │  API            │
│   Request)      │     │                  │     │                 │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                │                        │
                                │                        ▼
                                │                 ┌─────────────────┐
                                │                 │  User Device    │
                                │                 │  (Telegram App) │
                                │                 └─────────────────┘
                                │                        │
                                ▼                        ▼
                        ┌──────────────────────────────────────────┐
                        │  Decision Handler (receives callback)    │
                        │  Returns: allow/deny to Claude Code      │
                        └──────────────────────────────────────────┘
```

## Package Structure

```
src/
├── main.rs               # Entry point + tokio runtime
├── lib.rs                # Library root
├── cli.rs                # Clap subcommands (hook, stop, notify, event, bot, etc.)
├── config.rs             # JSON/env config loading (supports event_filters)
├── util.rs               # Shared utilities (read_stdin, project_name_from_cwd)
├── always_allow.rs       # Tool whitelist persistence
├── hook_handler.rs       # PermissionRequest handler (uses Messenger trait)
├── stop_handler.rs       # Stop hook - job completion notifications
├── notification_handler.rs # Notification hook handler
├── event_handler.rs      # Generic handler for all other hook events
├── bot.rs                # Long-running Telegram bot
├── telegram.rs           # Legacy re-exports for backward compatibility
├── error.rs              # Error types
└── messenger/            # Messenger abstraction layer
    ├── mod.rs            # Messenger trait + resolve_messenger()
    ├── types.rs          # Decision enum, PermissionMessage struct
    ├── telegram.rs       # Telegram implementation (inline keyboards)
    ├── discord.rs        # Discord implementation (buttons, requires --features discord)
    └── signal.rs         # Signal implementation (text-based, requires --features signal)
```

## Claude Code Hook Integration

Claude Code hooks are configured in `~/.claude/settings.json` or project's `.claude/settings.json`:
```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": { "tools": ["Bash", "Edit", "Write"] },
        "hooks": [{ "type": "command", "command": "claude-code-telegram hook" }]
      }
    ],
    "Stop": [
      {
        "matcher": {},
        "hooks": [{ "type": "command", "command": "claude-code-telegram stop" }]
      }
    ],
    "Notification": [
      {
        "matcher": {},
        "hooks": [{ "type": "command", "command": "claude-code-telegram notify" }]
      }
    ],
    "SessionStart": [
      {
        "matcher": {},
        "hooks": [{ "type": "command", "command": "claude-code-telegram event SessionStart" }]
      }
    ],
    "PostToolUseFailure": [
      {
        "matcher": {},
        "hooks": [{ "type": "command", "command": "claude-code-telegram event PostToolUseFailure" }]
      }
    ],
    "TaskCompleted": [
      {
        "matcher": {},
        "hooks": [{ "type": "command", "command": "claude-code-telegram event TaskCompleted" }]
      }
    ]
  }
}
```

Use `claude-code-telegram hooks-config` to generate a full hooks configuration for all supported events.

The hook script receives JSON via stdin with the permission request details and must output a JSON response.

## Development Commands

```bash
# Development build
cargo build

# Release build (~4MB)
cargo build --release

# Build with Discord support (~8MB)
cargo build --release --features discord

# Build with Signal support (~30MB)
cargo build --release --features signal

# Run tests
cargo test

# Run tests with Discord feature
cargo test --features discord

# Run tests with Signal feature
cargo test --features signal


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyujin-cho/claude-code-remote](https://github.com/kyujin-cho/claude-code-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

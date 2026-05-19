---
trigger: always_on
description: > Agent guidance for working effectively in this codebase.
---

# AGENTS.md — Stakpak CLI

> Agent guidance for working effectively in this codebase.

## Project Overview

Stakpak is a **security-hardened DevOps AI agent** that runs in the terminal. It generates infrastructure code, debugs Kubernetes, configures CI/CD, and automates deployments — without giving the LLM keys to production.

- **Language**: Rust (edition 2024, nightly features enabled)
- **License**: Apache-2.0
- **Repository**: https://github.com/stakpak/agent

## Workspace Structure

```
cli/                          # Main binary crate (`stakpak`)
├── src/
│   ├── main.rs
│   ├── commands/
│   │   ├── agent/run/        # Agent execution engine
│   │   │   ├── mode_interactive.rs   # Interactive TUI agent loop
│   │   │   ├── mode_async.rs         # Async/headless mode
│   │   │   ├── stream.rs             # SSE stream processing
│   │   │   ├── checkpoint.rs         # Session checkpoint/resume
│   │   │   ├── tooling.rs            # Tool execution
│   │   │   └── helpers.rs            # Shared helpers
│   │   ├── acp/              # Agent Client Protocol (Zed integration)
│   │   ├── mcp/              # MCP server/proxy commands
│   │   ├── auth/             # Login/account commands (interactive + non-interactive setup)
│   │   ├── autopilot/        # Autopilot: init, up/down, status, schedule, channel
│   │   └── watch/            # Scheduled task runtime (internal, driven by autopilot)
│   ├── config/               # Configuration management
│   │   ├── file.rs           # ConfigFile with profiles + ensure_readonly()
│   │   ├── profile.rs        # ProfileConfig + readonly_profile()
│   │   └── types.rs          # ProviderType (Remote/Local)
│   └── onboarding/           # Interactive setup wizard + save_config.rs
tui/                          # TUI crate (ratatui-based)
├── src/
│   ├── app/events.rs         # InputEvent / OutputEvent enums
│   └── services/handlers/    # Event handlers (tool, shell, etc.)
libs/
├── ai/                       # LLM provider abstraction (`stakai`)
│   └── src/providers/
│       ├── anthropic/        # Anthropic API (convert, stream, types)
│       ├── openai/           # OpenAI-compatible API
│       └── gemini/           # Google Gemini API
├── api/                      # API client + local processing (`stakpak-api`)
│   └── src/local/
│       ├── context_managers/ # Message history reduction strategies
│       │   ├── task_board_context_manager.rs   # Preserves individual messages
│       │   ├── simple_context_manager.rs       # Flattens history to text
│       │   └── file_scratchpad_context_manager.rs
│       └── hooks/            # Context hooks (scratchpad, task board)
├── shared/                   # Shared types (`stakpak-shared`)
│   └── src/models/
│       ├── llm.rs            # LLMMessage, LLMMessageContent, provider configs
│       ├── stakai_adapter.rs # ChatMessage → StakAI Message conversion
│       └── integrations/
│           ├── openai.rs     # ChatMessage, ToolCall, Role types
│           └── mcp.rs        # MCP tool call result handling
└── mcp/                      # MCP client/server/proxy crates
    ├── client/
    ├── server/
    └── proxy/
```

## Autopilot Architecture

The autopilot system (`stakpak autopilot` / `stakpak up`) is the self-driving infrastructure mode. It runs as a system service (launchd on macOS, systemd on Linux) and manages two runtimes:

### Config: `~/.stakpak/autopilot.toml`

Single config file for everything — schedules, channels, and runtime settings:

```toml
[runtime]
bind = "127.0.0.1:4096"

[[schedules]]
name = "health-check"
cron = "*/5 * * * *"
prompt = "Check system health"

[channels.slack]
bot_token = "xoxb-..."
app_token = "xapp-..."
```

### CLI Commands

```
stakpak up                              # Start autopilot (auto-inits if needed)
stakpak down                            # Stop autopilot
stakpak autopilot init                  # Explicit setup wizard
stakpak autopilot status                # Health, uptime, schedules, channels
stakpak autopilot logs                  # Stream logs
stakpak autopilot schedule list         # List schedules
stakpak autopilot schedule add <name> --cron '...' --prompt '...'
stakpak autopilot schedule remove <name>
stakpak autopilot schedule enable|disable <name>
stakpak autopilot schedule trigger <name>   # Manual fire
stakpak autopilot schedule history <name>
stakpak autopilot channel list          # List channels
stakpak autopilot channel add <type> --token|--bot-token|--app-token
stakpak autopilot channel remove <type>
stakpak autopilot channel test          # Test connectivity
```

### Key Files

| File | Purpose |
|------|---------|
| `cli/src/commands/autopilot.rs` | All autopilot commands, config types, schedule/channel CRUD |
| `cli/src/commands/watch/` | Schedule runtime (cron engine, trigger execution, history) |
| `libs/gateway/` | Channel runtime (Slack/Telegram/Discord message handling) |
| `libs/gateway/src/config.rs` | `GatewayConfig` — channel config load/save |

### Non-Interactive Setup (CI/scripts)

```bash
stakpak auth login --api-key $KEY
stakpak autopilot schedule add health --cron '0 */6 * * *' --prompt 'Check health'
stakpak autopilot channel add slack --bot-token $SLACK_BOT --app-token $SLACK_APP
stakpak up
```

## Architecture & Data Flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stakpak/agent](https://github.com/stakpak/agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

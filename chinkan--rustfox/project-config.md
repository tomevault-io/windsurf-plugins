---
trigger: always_on
description: RustFox is a Telegram AI assistant written in Rust. It connects to Telegram as a bot, uses OpenRouter LLM for inference (default model: `qwen/qwen3-235b-a22b`), provides built-in sandboxed tools (file I/O, command execution), and supports MCP (Model Context Protocol) servers for extensible tool integration. It implements an agentic loop that iterates tool calls until a final text response is produced (max iterations configurable, default 25).
---

# CLAUDE.md - RustFox Development Guide

## Project Overview

RustFox is a Telegram AI assistant written in Rust. It connects to Telegram as a bot, uses OpenRouter LLM for inference (default model: `qwen/qwen3-235b-a22b`), provides built-in sandboxed tools (file I/O, command execution), and supports MCP (Model Context Protocol) servers for extensible tool integration. It implements an agentic loop that iterates tool calls until a final text response is produced (max iterations configurable, default 25).

## Build & Run

```bash
# Build (debug)
cargo build

# Build (release)
cargo build --release

# Run (uses ./config.toml by default)
cargo run

# Run with custom config path
cargo run -- /path/to/config.toml

# Check without building
cargo check

# Format code
cargo fmt

# Lint
cargo clippy
```

### Configuration

Copy `config.example.toml` to `config.toml` and fill in credentials. The `config.toml` file is gitignored and must never be committed. Required fields:

- `telegram.bot_token` - Telegram Bot API token
- `telegram.allowed_user_ids` - Whitelist of Telegram user IDs
- `openrouter.api_key` - OpenRouter API key
- `sandbox.allowed_directory` - Directory for sandboxed file/command operations

### Home directory

RustFox stores all state under a single home directory (default `~/.rustfox`),
resolved as: `RUSTFOX_HOME` env (absolute) → `[general].home` config → `~/.rustfox`.
Layout: `config.toml`, `rustfox.db`, `skills/`, `agents/`, `workspace/` (the
sandbox), `artifacts/`, `user_model.md`. Each path can be pinned to an absolute
location in `config.toml`; unset paths fall back to the home default. Run
isolated instances with `RUSTFOX_HOME=...`. See
`docs/persistent-home-directory.md`. Path resolution lives in `src/home.rs`
(`Config::resolve` writes the resolved absolute paths back into the config).
Bundled skills/agents are seed-copied on first run; `/update-skills` re-syncs
them using `<home>/skills-lock.json`.

## Architecture

```
src/
├── main.rs      # Entry point: logging init, config loading, MCP setup, bot launch
├── config.rs    # TOML config parsing (Config, TelegramConfig, OpenRouterConfig, SandboxConfig, McpServerConfig)
├── llm.rs       # OpenRouter API client (ChatMessage, ToolCall, ToolDefinition, LlmClient)
├── tools.rs     # Built-in tool definitions and execution with sandbox path validation
├── mcp.rs       # MCP client manager (McpManager, McpConnection) for external tool servers
└── bot.rs       # Telegram bot handler: message routing, agentic loop, conversation state
```

### Data Flow

1. User sends a Telegram message
2. `bot.rs` filters by `allowed_user_ids`, routes commands (`/start`, `/clear`, `/tools`)
3. Non-command messages enter `process_with_llm()` which runs the agentic loop
4. `llm.rs` sends conversation history + tool definitions to OpenRouter
5. If LLM returns tool calls, `execute_tool()` dispatches to built-in tools or MCP tools
6. Tool results are appended to conversation and the loop repeats (up to 10 iterations)
7. Final text response is split into <=4000 char chunks and sent back via Telegram

### Key Components

- **AppState** (`bot.rs`): Shared state holding `LlmClient`, `Config`, `McpManager`, and per-user `Conversation` map behind a `Mutex`
- **LlmClient** (`llm.rs`): Stateless HTTP client for OpenRouter's `/chat/completions` endpoint with tool-calling support
- **McpManager** (`mcp.rs`): Manages stdio-based MCP server child processes. Tools are namespaced as `mcp_{server_name}_{tool_name}`
- **Sandbox validation** (`tools.rs`): All file/command operations are restricted to the configured sandbox directory via path canonicalization

## Code Conventions

### Rust Patterns

- **Edition**: 2021
- **Async runtime**: Tokio with `full` features
- **Error handling**: `anyhow::Result` throughout, with `.context()` / `.with_context()` for error messages
- **Logging**: `tracing` crate with `tracing-subscriber` (env filter: `RUST_LOG`, default `info,rustfox=debug`)
- **Serialization**: `serde` derive macros with `#[serde(skip_serializing_if = "Option::is_none")]` for optional fields
- **Shared state**: `Arc<AppState>` passed via teloxide's dependency injection (`dptree::deps!`)
- **Concurrency**: `tokio::sync::Mutex` for per-user conversation map (not `std::sync::Mutex`)

### Naming

- Module names are single words (`bot`, `config`, `llm`, `mcp`, `tools`)
- Struct fields use `snake_case`
- JSON field renames use `#[serde(rename = "type")]` where the Rust field name differs from the API field

### Error Handling Style

- Use `anyhow::bail!()` for early returns with error messages
- Use `.context("message")` on `Result` chains for context propagation
- MCP connection failures are logged but do not abort startup (`connect_all` catches errors)
- Tool execution errors return error strings to the LLM rather than crashing

### Security


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chinkan/RustFox](https://github.com/chinkan/RustFox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

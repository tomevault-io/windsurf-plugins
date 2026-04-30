---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
cargo build --workspace          # Build all crates
cargo test --workspace           # Run all tests
cargo test -p octos-agent         # Test single crate
cargo test -p octos-agent test_name  # Run single test
cargo clippy --workspace         # Lint
cargo fmt --all                  # Format
cargo fmt --all -- --check       # Check formatting
cargo install --path crates/octos-cli  # Install CLI locally
```

## Architecture

octos is a Rust-native, API-first Agentic OS — multi-tenant AI agent platform. 8-crate workspace + bundled skills, layered:

```
octos-cli  (CLI: clap commands, config loading, config watcher)
    |
octos-agent  (Agent loop, tool system, sandbox, MCP, compaction, plugins)
    |          \
octos-memory   octos-llm  (hybrid search + memory store | LLM providers)
    \           /
    octos-core  (Task, Message, Error types, truncate_utf8 - no internal deps)
```

Alongside octos-agent:
- **octos-bus**: Message bus, 14 channels (Telegram/Discord/Slack/WhatsApp/Email/WeChat/...), sessions, coalescing, cron, heartbeat
- **octos-pipeline**: DOT-graph pipeline engine — per-node model selection, parallel fan-out, checkpoints, human gates
- **octos-plugin**: Plugin SDK — manifest parsing, discovery, gating (binary/env/OS checks)

Bundled skills in `crates/app-skills/` (weather, time, news, deep-search, etc.) and `crates/platform-skills/` (voice).

Commands: chat, init, status, gateway, serve, clean, completions, cron, channels, auth (login/logout/status), skills (list/install/remove).

Three runtime modes: `octos chat` (interactive CLI), `octos gateway` (multi-channel), `octos serve` (web dashboard + 91 REST endpoints).

Auth module (`octos-cli/src/auth/`): OAuth PKCE + device code for OpenAI, paste-token for others. Stored in `~/.octos/auth.json`. `config.rs` checks auth store before env vars.

### Key Flow: Agent Loop (`octos-agent/src/agent.rs`)

1. Build messages (system prompt + conversation history + memory context)
2. Call LLM with tool specs (filtered by ToolPolicy + provider policy)
3. If tool calls returned -> execute tools -> append results -> loop
4. If EndTurn or budget exceeded -> return result
5. Context compaction kicks in when token budget fills (`compaction.rs`)

### Tool System (`octos-agent/src/tools/`)

All tools implement `Tool` trait (`spec() -> ToolSpec`, `execute(&Value) -> ToolResult`). Registered in `ToolRegistry` (HashMap). Tools: shell, read_file, write_file, edit_file, glob, grep, list_dir, web_search, web_fetch, message, spawn, cron, browser (feature-gated). Tool argument size limit: 1MB (non-allocating `estimate_json_size` with escape accounting). File tools use `O_NOFOLLOW` (Unix) for symlink-safe I/O. Shared SSRF protection in `tools/ssrf.rs`.

**Tool Policies** (`tools/policy.rs`): Allow/deny lists with deny-wins semantics, wildcard matching (`exec*`), and named groups (`group:fs`, `group:runtime`, `group:search`, `group:web`, `group:sessions`). Provider-specific policies via `tools.byProvider` in config.

### Sandbox (`octos-agent/src/sandbox.rs`)

Three sandbox backends: `Bwrap` (Linux), `Macos` (sandbox-exec), `Docker`. On Windows, falls back to `NoSandbox` (uses `cmd /C`) or Docker if available. Auto-detection in `SandboxMode::Auto`. Shared `BLOCKED_ENV_VARS` constant (18 env vars) across all backends and MCP server spawning. Docker supports mount modes (none/ro/rw), resource limits (CPU/memory/PIDs), network isolation. Path validation rejects injection characters (`:`, `\0`, `\n`, `\r` for Docker; control chars, `(`, `)`, `\`, `"` for macOS SBPL).

### MCP (`octos-agent/src/mcp.rs`)

JSON-RPC stdio transport for MCP servers. Env var sanitization via shared `BLOCKED_ENV_VARS`. Input schema validation: max depth 10, max size 64KB — tools with invalid schemas are rejected at registration.

### Context Compaction (`octos-agent/src/compaction.rs`)

Token-aware message compaction: estimates tokens, strips tool arguments, summarizes to first lines, preserves recent tool call/result pairs.

### LLM Providers (`octos-llm/src/`)

`LlmProvider` trait with `chat()` method. Four native providers: `AnthropicProvider`, `OpenAIProvider`, `GeminiProvider`, `OpenRouterProvider`. 8 OpenAI-compatible via `with_base_url()`. 3-layer failover: `RetryProvider` (exponential backoff on 429/5xx) → `ProviderChain` → `AdaptiveRouter` (hedge racing, lane scoring, circuit breakers).

### Plugin System (`octos-agent/src/plugins/`, `octos-plugin/`)

Skills are self-contained binaries with `manifest.json` declarations. Binary protocol: `./skill_binary <tool_name>` with JSON on stdin, JSON `{success, output, files_to_send}` on stdout. Discovery scans directories with precedence rules. Gating checks binary existence, env, and OS requirements.

**spawn_only tools**: Manifest field `spawn_only: true` marks tools for background execution. Auto-intercepted in the execution loop — wrapped in `tokio::spawn`, returns immediately. No LLM cooperation needed. SKILL.md auto-injected as system prompt for skills with spawn_only tools.

### Pipeline Engine (`octos-pipeline/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [octos-org/octos](https://github.com/octos-org/octos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

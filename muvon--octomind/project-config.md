---
trigger: always_on
description: Session-based AI assistant where the model calls MCP tools (read/write files, search, shell, delegate) to do real work. Sessions run interactively (CLI), non-interactively (`--format`), or as daemons (ACP/WebSocket). Config is the single source of truth — all runtime behavior (model, tools, roles, compression, learning) derives from TOML. Multi-provider via `octolib`. Rust 1.95+, tokio async, `clap` CLI.
---

# Octomind — AI Development Assistant (Rust)

Session-based AI assistant where the model calls MCP tools (read/write files, search, shell, delegate) to do real work. Sessions run interactively (CLI), non-interactively (`--format`), or as daemons (ACP/WebSocket). Config is the single source of truth — all runtime behavior (model, tools, roles, compression, learning) derives from TOML. Multi-provider via `octolib`. Rust 1.95+, tokio async, `clap` CLI.

## Project Structure

```
src/
  main.rs              # Entry: CLI parsing → Config::load() → subcommand dispatch
  lib.rs               # Spinner-aware print macros (shadow std::println! etc.)
  config/              # Config types, loading, migrations, log macros
  mcp/                 # Tool routing, server lifecycle, all builtin tools
    core/              # plan, local_tool, functions
    orchestration/     # tap, schedule
    runtime/           # mcp, agent, skill, capability tools
    agent/             # agent_* tool routing → layer/subprocess
  session/
    chat/session/      # ChatSession: init, main loop, command dispatch, API calls
    chat/              # Response processing, tool execution, compression, display
    context.rs         # Session-scoped state (task-local SessionId propagation)
    layers/            # AI sub-agent execution
    guardrails.rs      # Guardrails (pipe) loading and evaluation
    pipe.rs             # Pipe execution logic
    workflows/         # AI-orchestrated multi-step workflows
    learning/          # Cross-session lesson extraction/injection
  acp/                 # ACP stdio server (agent-to-agent protocol)
  websocket/           # WebSocket server for remote sessions
  agent/               # Tap registry, manifest resolution, dependency resolution
  commands/            # CLI subcommand handlers
config-templates/
  default.toml         # ALL config fields with defaults — single source of truth
  agents/              # Agent template files
```

## Where to Look First

| Task | Start here |
|------|------------|
| Add a new MCP tool | `src/mcp/core/functions.rs` (core) or `src/mcp/runtime/mod.rs` (runtime) → then route in `src/mcp/mod.rs` |
| Add a session command (`/foo`) | `src/session/chat/session/commands/` → `mod.rs` → `src/session/chat/commands.rs` |
| Change a config field/default | `config-templates/default.toml` first, then matching type in `src/config/` |
| Trace a config load failure | `src/config/loading.rs` → `load()` |
| Understand MCP server activation | `src/config/mod.rs` → `get_merged_config_for_role()` |
| Debug tool not found/routing | `src/mcp/tool_map.rs` → `get_server_for_tool()`, then `src/mcp/mod.rs` → `try_execute_tool_call()` |
| Session init / state management | `src/session/context.rs` → `init_session_services()` |
| Session main loop | `src/session/chat/session/main_loop.rs` |
| Response / tool execution flow | `src/session/chat/response.rs` → `src/session/chat/response/tool_execution.rs` |
| Skill auto-activation | `src/mcp/core/skill_auto.rs` |
| Layer / guardrails / workflow | `src/session/layers/`, `src/session/guardrails.rs`, `src/session/pipe.rs`, `src/session/workflows/` |
| Learning system | `src/learning/` |
| ACP server | `src/acp/agent.rs` |
| Sandbox | `src/sandbox/mod.rs` |
| Directory path constants | `src/directories.rs` |

## Architecture: The Flows That Matter

### Config → Role → Tools (activation chain)

```
Config::load()
  └─ merge all *.toml in config_dir (alphabetical)
     then mcp-*.toml files AFTER base files (override same-named servers)
     arrays: concat + dedup by `name`; tables: deep-merge

get_merged_config_for_role(role)            [src/config/mod.rs]
  └─ collects servers: explicit server_refs UNION auto_bind matches
     auto_bind matches on EXACT string — "developer" ≠ "developer:general"
  └─ result: merged config with only this role's servers visible

initialize_mcp_for_role(role, merged_config)
  └─ spawns stdio / opens http / registers builtins
  └─ builds TOOL_MAP: tool_name → McpServerConfig

try_execute_tool_call(call)                 [src/mcp/mod.rs]
  └─ TOOL_MAP lookup → routes to: core | runtime | agent | local | external
```

**Key rules:**
- `mcp-*.toml` always loads after `mcp.toml` regardless of filename sort order — use this for overrides
- `mcp persist` writes `<config_dir>/mcp-<name>.toml` with `auto_bind = ["<role>"]` — picked up on next start
- `allowed_tools` non-empty → silently filters tools not listed; `get_merged_config_for_role` auto-appends `"<server>:*"` for auto-bind servers to prevent accidental filtering

### Session Lifecycle (CRITICAL INVARIANT)

Five entry points all share the same initialization contract. When adding session-scoped state, ALL five must be updated:

| Mode | Entry point |
|------|-------------|
| Interactive + non-interactive CLI | `src/session/chat/session/main_loop.rs` → `init_session_runtime()` |
| ACP new_session | `src/acp/agent.rs` ~line 568 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Muvon/octomind](https://github.com/Muvon/octomind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

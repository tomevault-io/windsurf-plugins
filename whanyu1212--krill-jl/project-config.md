---
trigger: always_on
description: - This file is the root onboarding guide for Codex and other coding agents working in this repository.
---

# Repository Guidelines

## Purpose
- This file is the root onboarding guide for Codex and other coding agents working in this repository.
- Prefer reading the current code over inferring architecture from stale comments. `src/Krill.jl` is the authoritative include order for the package.
- Treat this repo as a Julia-native agent runtime, not a thin bot wrapper: most changes touch prompt assembly, tool dispatch, persistence, or channel/runtime wiring.

## Start Here
- `README.md` explains the product surface and top-level config shape.
- `src/Krill.jl` shows module load order and public exports.
- `src/runtime.jl` shows how channels, session processing, tools, MCP, cron, memory, and subagents are wired into `RuntimeState`.
- `src/agent.jl` defines the configuration surface that feature work usually extends: `MemoryConfig`, `BuiltinToolsConfig`, `SkillsConfig`, `ClaudeCodeConfig`, `CodexConfig`, `PromptContextConfig`, `SubagentConfig`, and `RetryConfig`.
- `src/prompt_context.jl` shows how system instructions are composed from bootstrap docs, skill metadata, always-on skills, session memory, global memory, tool-safety guidance, and runtime metadata.
- `test/runtests.jl` is the map of what the suite actually covers.

## Architecture Map
- `bin/krill.jl` is the runtime entry point.
- `src/Krill.jl` is the package entry point and include-order source of truth. Include order matters in this codebase.
- `src/transport/` owns normalized message types and queueing:
  - `types.jl`
  - `message_hub.jl`
  - `manager.jl`
  - `dedup.jl`
  - `channels.jl`
  - `durable_queue.jl`
- `src/sessions/` owns persistence and long-lived conversation state:
  - `sessions.jl` for history persistence
  - `memory.jl` for session memory files
  - `memory_consolidation.jl` for LLM-driven session-memory summarization
  - `global_memory.jl` for user-wide cross-session memory
  - `consumer.jl` for per-session FIFO processing and cancellation
  - `echo.jl` for the non-LLM processor path
- `src/tools/` owns the tool system:
  - `registry.jl` for `ToolDef`, `ToolRegistry`, and dispatch
  - `skills.jl` for workspace and builtin `SKILL.md` discovery
  - `mcp.jl` for MCP client registration and namespacing
  - `builtin/` for local tools: file, web, shell, GitHub, Google Workspace, cron, message, Claude Code, and Codex
- `src/llm/` owns provider integration and the tool loop:
  - `providers.jl`, `api.jl`, `chat_completion.jl`
  - `parsing.jl` for provider payload conversion
  - `tool_loop.jl` for iterative tool execution
  - `processor.jl` for session processor construction
  - `context.jl` and `llm.jl` for provider-facing assembly
- `src/scheduling/` owns background execution:
  - `cron.jl`
  - `subagent.jl`
- `src/channels/` owns Telegram and Discord adapters.
- `src/config/` owns config parsing, environment expansion, provider creation, MCP config, and channel construction.
- `src/runtime.jl` wires everything together into `RuntimeState`.

## Runtime Flow
- Inbound channel payloads are normalized into `InboundMessage`.
- `BoundedDedup` suppresses repeat deliveries before they hit the hub.
- `MessageHubState` feeds the session consumer.
- `run_session_loop!` enforces per-session FIFO while allowing concurrent sessions.
- The processor is either echo or an LLM-backed processor from `make_llm_processor`.
- LLM turns may call local builtins, MCP tools, cron tools, Claude Code, or Codex through the tool loop.
- Turn history is persisted to the session store, and outbound messages are dispatched through `ChannelManagerState`.
- Cron jobs inject synthetic inbound messages; subagents run background tasks with their own bounded iteration limits.

## Prompt And Memory Model
- Prompt composition lives in `src/prompt_context.jl`; do not collapse it back into one static prompt string.
- Bootstrap docs are loaded from the workspace in this order: `AGENTS.md`, `SOUL.md`, `USER.md`, `TOOLS.md`.
- Missing bootstrap docs are skipped, not treated as errors.
- Skills come from `context/skills/` plus optional builtin skill directories. Workspace skills override builtin skills with the same name.
- Skills with `always: true` frontmatter are injected into every turn if their requirements are met.
- The composed instruction stack can include:
  - base system prompt
  - workspace bootstrap docs
  - available-skills summary
  - always-on skill content
  - global user memory
  - per-session memory
  - tool-output safety guidance
  - runtime metadata such as UTC timestamp, channel, session key, chat id, and user id
- Session history persists under `<data_dir>/sessions/<session>/history.jsonl`.
- Session memory persists under `<data_dir>/memory/<session>/` as `MEMORY.md`, `HISTORY.md`, and `state.json`.
- Global memory persists under `<data_dir>/global_memory/<user_id>/MEMORY.md`.
- Cron jobs persist under `<data_dir>/cron/jobs.json`.
- Dead letters persist under `<data_dir>/dead_letters.jsonl`.

## Project Structure And Files To Touch
- Add or change tool behavior in `src/tools/builtin/*.jl` and check registration in `src/tools/builtin/registration.jl`.
- Add MCP behavior in `src/tools/mcp.jl`. MCP tools are namespaced as `mcp_<server>_<tool>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whanyu1212/Krill.jl](https://github.com/whanyu1212/Krill.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->

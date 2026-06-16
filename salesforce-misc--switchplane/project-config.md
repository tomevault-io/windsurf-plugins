---
trigger: always_on
description: Switchplane is a Python runtime control plane for agent-based task execution. It is LangGraph-native — tasks are defined as LangGraph StateGraph graphs. Each app built with Switchplane becomes a standalone CLI with its own isolated daemon and runtime directory.
---

# CLAUDE.md — Switchplane

## What is this project?

Switchplane is a Python runtime control plane for agent-based task execution. It is LangGraph-native — tasks are defined as LangGraph StateGraph graphs. Each app built with Switchplane becomes a standalone CLI with its own isolated daemon and runtime directory.

## Architecture in one sentence

Each app gets its own CLI that sends requests over a Unix socket to a per-app daemonized control plane, which spawns agent subprocesses that communicate bidirectionally over a per-agent Unix socketpair using length-prefixed JSON framing.

## Project layout

```
src/switchplane/           # Main package (pip-installable)
  __init__.py              # Public API re-exports: Application, Shell, Task, command, Field
  app.py                   # Application container and McpServerConfig
  shell.py                 # Sandboxed subprocess execution with path/command allowlists, LangChain tool factory
  agent.py                 # AgentSpec and AgentRecord models
  task.py                  # Task ABC, TaskRecord, TaskStatus, @command decorator, parameter introspection
  protocol.py              # IPC message types: CliRequest/Response, AgentEvent/Command
  transport.py             # Unix socket server (async) + client (sync), 4-byte length-prefixed framing
  persistence.py           # SQLite Store (aiosqlite, WAL mode) — tasks, agents, events tables
  checkpoint.py            # LangGraph BaseCheckpointSaver backed by SQLite
  discovery.py             # Agent/task discovery via module import (no entry point discovery)
  daemon.py                # Daemonization (double-fork), signal handling, idle timer, RuntimePaths
  control_plane.py         # Central orchestrator — request dispatch, single app, idle shutdown
  subprocess_manager.py    # Agent subprocess lifecycle — socketpair creation, event reading, command sending
  agent_runtime.py         # Runs inside agent subprocess — AgentContext, bidirectional IPC over socketpair
  cli.py                   # Click CLI factory — build_cli(app) generates run, runtime, agent, task command groups
  tui.py                   # prompt_toolkit full-screen TUI — tab-based task viewer with system tab [0]
  fmt.py                   # Shared formatting utilities (e.g. tree rendering for detail payloads)
  config.py                # TOML config loading — cascading: app defaults + user overrides

  mcp.py                   # MCP client lifecycle: McpSession, McpManager, LangChain tool wrapper

  _util.py                 # Shared constants (MAX_MESSAGE_SIZE)
  llm.py                   # LLM provider routing (ChatAnthropic/OpenAI/Google via model prefix)
  logging.py               # structlog configuration
  oauth.py                 # OAuth client for MCP HTTP servers (PKCE flows)
  scaffold.py              # `switchplane init` project scaffolding

examples/hello/        # Simple LangGraph graph (get_user → say_hello)
examples/weather/      # Long-running polling task (Open-Meteo weather watch, @command for coordinates)
examples/devops/       # Ops review: deterministic pandas analysis + LLM summary
examples/chatbot/      # Interactive LLM chat with interrupts
tests/                     # Test directory (pytest)
```

## Key design rules

- **The control plane owns task/event persistence.** Agents write only checkpoint data via a separate WAL-mode connection.
- **The control plane never runs domain logic.** All user code runs inside agent subprocesses.
- **Tasks are first-class.** Agents are just execution hosts. Every task has an ID, status, events, and stored results.
- **LangGraph-native.** Do not introduce generic workflow abstractions. Tasks use LangGraph StateGraph directly.
- **One app per runtime.** Each Application gets its own daemon, database, and socket.

## Application entry point

`Application(name="myapp")` creates an app. `app.discover_agents("myapp.agents")` registers discovery roots. `app.run()` discovers agents, builds the CLI, and starts it. The `name` determines the runtime directory (`~/.myapp/`). The CLI entry point is registered via `[project.scripts]` in pyproject.toml.

## Configuration

Two-layer cascading TOML config. App-bundled defaults (specified via `Application(default_config=Path(...))`) are deep-merged with user-level overrides at `~/.{app_name}/config.toml`. User config wins on conflict. Apps ship base URLs, model defaults, and agent settings; users provide personal config like API keys. Pydantic model: `AppConfig` with `LLMConfig` and per-agent dicts. Config is passed to agent subprocesses via the `execute_task` command payload and available as `ctx.config` in the agent runtime.

## How to run

```bash
# Setup
uv venv .venv && source .venv/bin/activate
uv pip install -e . -e examples/hello -e examples/weather -e examples/devops -e examples/chatbot

# Run a task (streams events inline, Ctrl+C detaches)
hello run example hello --name Alice

# Run detached
weather run weather watch -d

# Bare app invocation opens the full-screen TUI (auto-discovers running tasks)
hello

# Operator commands

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salesforce-misc/switchplane](https://github.com/salesforce-misc/switchplane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

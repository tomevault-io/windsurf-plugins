---
trigger: always_on
description: Motus is a full-stack AI agent framework built on a custom async task-graph runtime. It provides agent definitions (ReAct loop), multi-provider LLM clients, a composable tool system (function tools, MCP, Docker sandboxes), persistent memory, a skills system, and FastAPI-based serving infrastructure.
---

# Motus — Agent Guide

Motus is a full-stack AI agent framework built on a custom async task-graph runtime. It provides agent definitions (ReAct loop), multi-provider LLM clients, a composable tool system (function tools, MCP, Docker sandboxes), persistent memory, a skills system, and FastAPI-based serving infrastructure.

## Commands

```bash
# Install all dependencies (core + dev)
uv sync --all-extras --dev

# Run unit tests (fast, no API keys needed)
uv run pytest -v tests/unit -m "not slow"

# Run integration tests with VCR replay (no API keys needed)
uv run pytest -v tests/integration -m "integration"

# Re-record VCR cassettes (requires real API keys in .env)
uv run pytest tests/integration/examples/ -v --vcr-record=all

# Format code
uv run ruff format

# Lint and auto-fix
uv run ruff check --fix

# Run all pre-commit hooks on all files
uv run pre-commit run --all-files

# Install git hooks (first-time setup)
uv run pre-commit install

# Start the agent server
uv run motus-serve start myapp:server --port 8000

# Interactive chat with an agent
uv run motus-serve chat
```

## Tech Stack

- **Python** 3.12+ (uses modern syntax: `X | None`, `dict[K, V]`, `TYPE_CHECKING`)
- **Async runtime**: asyncio + `concurrent.futures` for cross-thread coordination
- **Validation**: pydantic >= 2.12.5
- **Web**: fastapi >= 0.128.0, uvicorn >= 0.40.0, httpx >= 0.28.1
- **LLM clients**: anthropic >= 0.40.0, openai >= 2.15.0, google-genai >= 1.0.0
- **Tool execution**: docker >= 7.1.0, kubernetes >= 34.1.0, mcp[cli] >= 1.25.0, paramiko >= 4.0.0
- **Testing**: pytest >= 9.0.2, pytest-asyncio >= 0.24.0, vcrpy >= 8.1.1
- **Linting**: ruff (rules: E, F, I, W; ignores: E501, E402)
- **Package manager**: uv (workspace with members `.` and `services/*`)

## Project Structure

```
src/motus/
├── __init__.py              # Motus, ModelClient — high-level API
├── runtime/                 # Core task-graph engine (owner: @NorthmanPKU)
│   ├── agent_runtime.py     #   GraphScheduler (event loop, dependency DAG, retries/timeouts)
│   │                        #   AgentRuntime (thread-safe wrapper, cross-thread task submission)
│   ├── agent_task.py        #   @agent_task decorator, AgentTaskDefinition (descriptor protocol)
│   ├── agent_future.py      #   AgentFuture — lazy future with operator overloading & sync barriers
│   ├── task_instance.py     #   TaskInstance (COMPUTE/RESOLVE), TaskPolicy, TaskStatus, stack stitching
│   ├── hooks.py             #   HookManager — task_start / task_end / task_error lifecycle events
│   ├── types.py             #   AgentTaskId, AgentFutureId counters
│   └── tracing/             #   Distributed tracing, OpenTelemetry export, live trace server
├── agent/                   # Agent definitions (owners: @NorthmanPKU, @JackFram)
│   ├── base_agent.py        #   AgentBase[T] — abstract base with tools, memory
│   ├── react_agent.py       #   ReActAgent — Reasoning + Acting loop
│   ├── skills.py            #   Skill loading (SKILL.md) and tool creation
│   └── tasks.py             #   model_serve_task — agent task for model calls
├── tools/                   # Tool ecosystem (owners: @eliotsolomon18, @coppock, @NorthmanPKU)
│   ├── core/
│   │   ├── tool.py          #   Tool protocol (description, json_schema, __call__)
│   │   ├── function_tool.py #   FunctionTool — wraps Python functions as LLM tools
│   │   ├── mcp_tool.py      #   MCP (Model Context Protocol) tool integration
│   │   ├── sandbox.py       #   Sandbox ABC (create/acreate/connect factory methods)
│   │   ├── tool_provider.py #   ToolProvider interface
│   │   ├── composite_tool_provider.py
│   │   ├── normalize.py     #   Schema generation and tool normalization
│   │   └── decorators.py    #   @tool and @tools convenience decorators
│   ├── providers/
│   │   ├── docker/          #   DockerSandbox — container-based code execution
│   │   └── brave/           #   Brave Search tool
│   └── runtime/
│       └── sync_manager.py  #   Sync/async coordination for tool calls
├── models/                  # Multi-provider LLM clients (owner: @yzhou442)
│   ├── base.py              #   BaseChatClient, ChatMessage, ToolCall, ChatCompletion
│   ├── anthropic_client.py  #   Claude API
│   ├── openai_client.py     #   OpenAI API
│   ├── gemini_client.py     #   Google Gemini API
│   ├── openrouter_client.py #   OpenRouter aggregator
│   └── message_schema.py    #   Unified message data models
├── memory/                  # Memory & context management (owners: @JackFram, @vasiliskyp)
│   ├── base_memory.py       #   BaseMemory abstract interface
│   ├── basic_memory.py      #   BasicMemory (simple in-memory)
│   ├── compaction_memory.py #   CompactionMemory (auto-compacting context)
│   ├── config.py            #   CompactionMemoryConfig
│   ├── interfaces.py        #   ConversationLogStore ABC
│   └── stores/              #   LocalConversationLogStore
├── serve/                   # Agent serving infrastructure
│   ├── server.py            #   AgentServer (FastAPI)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lithos-ai/motus](https://github.com/lithos-ai/motus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->

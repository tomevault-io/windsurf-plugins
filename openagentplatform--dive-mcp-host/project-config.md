---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Role

This repo is `dive-mcp-host` — the Python MCP host service. It is consumed
two ways:
- as a **standalone HTTP service** (`dive_httpd`) and CLI (`dive_cli`)
- as the **Python submodule** of the Dive desktop app (Electron + Tauri),
  which spawns this service as a child process

Backend changes that touch the wire format must stay backwards-compatible
with whatever Dive frontend is in the field. See
`memory/feedback_backcompat.md` for the discipline.

## Common Commands

```bash
# Install runtime + dev deps (uv is preferred)
uv sync --extra dev --frozen
# fallback: pip install -e ".[dev]"

# Start HTTP service on 0.0.0.0:61990
dive_httpd

# One-shot CLI chat
dive_cli "hello"
dive_cli -c CHAT_ID "follow-up"   # resume a thread

# Tests
pytest                                              # full suite
pytest tests/test_tools.py::test_mcp_server_info    # single test
pytest -m integration                               # integration only
uv run --extra dev --frozen pytest                  # without activating venv

# Lint / format (ruff is the only linter; ruff format is the formatter)
ruff check .
ruff format .

# Local Postgres for tests/dev (optional)
./scripts/run_pg.sh

# DB migrations (alembic.ini → ./dive_mcp_host/httpd/database/migrations/)
alembic upgrade head
alembic revision --autogenerate -m "<message>"
```

### Running tests in this sandbox

Tests subprocess-spawn `python3` from `PATH`. If running outside the
project venv, prepend it explicitly so the child processes see the
installed deps:

```bash
PATH="$PWD/.venv/bin:$PATH" \
PYTHONPATH=/path/to/extra/deps:$PWD \
.venv/bin/python -m pytest tests/...
```

## Architecture

### Layered structure

```
dive_mcp_host/
├── host/         ← core SDK (no HTTP, embeddable)
├── httpd/        ← FastAPI server wrapping the SDK
├── cli/          ← dive_cli entry point
├── models/       ← LLM provider loaders
├── skills/       ← skill (markdown-with-frontmatter) loader
├── plugins/      ← plugin discovery
├── oap_plugin/   ← OpenAgentPlatform integration
├── internal_tools/ ← built-in non-MCP tools (fetch, bash, file IO)
└── scheduler/    ← background scheduling
```

### Core abstractions (everything is an async context manager)

The SDK is built on `host/helpers/context.py:ContextProtocol`. Every
long-lived resource implements `_run_in_context() -> AsyncGenerator[Self, None]`
and is used via `async with`. Lifecycle is strictly nested.

```
DiveMcpHost                     host/host.py
  ├── ToolManager               host/tools/__init__.py
  │     └── McpServer × N       host/tools/mcp_server.py
  │           └── ServerSessionStore (per chat_id)
  ├── OAuthManager              host/tools/oauth.py
  ├── ElicitationManager        host/tools/elicitation_manager.py
  ├── ToolManagerPlugin         host/tools/plugin.py  (non-MCP tools)
  ├── checkpointer (optional)   langgraph SqliteSaver/PostgresSaver
  └── Chat                      host/chat.py  (per conversation)
        └── agent               host/agents/* (langgraph compiled graph)
```

`DiveMcpHost.chat(...)` returns a `Chat` context which compiles a langgraph
agent (default: `agents/chat_agent.py`) over the tools collected from
`ToolManager`. Variants: `tools_in_prompt` (Ollama-friendly),
`message_order`, `file_in_additional_kwargs`. Selection happens via the
`get_agent_factory_method` parameter.

### MCP server connection (`McpServer`)

Single `McpServer` covers all transport types via dispatch in `__init__`:
- `command + transport=stdio` → `_stdio_setup` / `_stdio_session`
- `command + url` → `_local_http_setup` (spawn process, then connect SSE)
- `url only` → `_http_setup` (`sse`, `streamable`, `websocket`)

Initialization (`_init_tool_info`):
1. Call `session.initialize()` and read advertised `capabilities`.
2. Only call `tools/list` if `capabilities.tools is not None`.
3. Only call `prompts/list` if `capabilities.prompts is not None`.
4. Both calls go through `_safe_list`, which swallows `-32601 Method not
   found` so a misbehaving server cannot break startup.

State machine (`ClientState`): `INIT → RUNNING / FAILED → RESTARTING →
CLOSED`. Transitions go through `__change_state` under `self._cond`;
waiters use `wait([states])`.

Session reuse: `ServerSessionStore` keys on `chat_id` so multiple tool
calls in the same chat share a session; new chats spawn new sessions.

### HTTP layer (`httpd/`)

`httpd/server.py:DiveHostAPI(FastAPI)` holds a `dive_host` dict keyed by
host name (default `"default"`). Routers live in `httpd/routers/`:

| Prefix | Router | Notes |
| --- | --- | --- |
| `/api/chat` | `chat.py` | Streaming chat, message store integration |
| `/api/v1/mcp` | `chat.py` (re-mount) | Remote MCP-style endpoints |
| `/api/tools` | `tools.py` | Server list, prompts API, OAuth, elicitation, log stream |
| `/api/config` | `config.py` | Reload/inspect MCP + model config |
| `/api/skills` | `skills.py` | Skill listing |
| `/v1/openai` | `openai.py` | OpenAI-compatible endpoint |
| `/model_verify` | `model_verify.py` | One-shot LLM credential check |

OAuth callback HTML lives in `httpd/templates/oauth_callback.html`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenAgentPlatform/dive-mcp-host](https://github.com/OpenAgentPlatform/dive-mcp-host) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

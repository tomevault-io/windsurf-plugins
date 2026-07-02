---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A control layer for the [Reachy Mini](https://github.com/pollen-robotics/reachy_mini) robot. Nothing here drives motors directly — every action is an HTTP call to the **Reachy Mini daemon** (default `http://localhost:8000`), which must be running separately. The code lives in the **`reachy_mini_mcp/`** package (a pip-installable distribution, `reachy-mini-mcp`, built with hatchling). It exposes the daemon's capabilities to LLMs two ways, both backed by one shared tool repository:

- **`reachy_mini_mcp/server.py`** — a FastMCP (stdio) MCP server. Exposes exactly **one** MCP tool, `operate_robot`, a meta-tool that dispatches to every robot operation by name. Individual tools are loaded into an in-process registry but deliberately *not* surfaced as separate MCP tools.
- **`reachy_mini_mcp/server_openai.py`** — a FastAPI HTTP server (port **8100**) that speaks an OpenAI-ish dialect: `GET /tools`, `POST /execute_tool`, `POST /v1/chat/completions`. Here each tool *is* exposed individually in OpenAI function-calling format.

In front of both sits **`reachy_mini_mcp/cli/`** — the `reachy-mini-mcp` console
script, an MCP-server **manager**: `overview`, `show` (print the mcp.json
snippet), `explain`, `install`/`uninstall` (merge/remove the entry in a client
config), `doctor`, and `serve` (run the server — what mcp.json launches). The
manager imports **no** robot dependencies; only `serve` pulls in the FastMCP
stack (lazily), so a bare `pip install reachy-mini-mcp` is a working manager and
the robot runtime lives behind the `[server]` / `[tts]` / `[openai]` extras.

```text
MCP client / HTTP client  →  reachy_mini_mcp.server | .server_openai  →  Reachy daemon (:8000)  →  robot/sim
                  manager:  reachy-mini-mcp {show,install,doctor,serve,...}
```

Beyond the robot-control layer, this repo is also an **AgentCulture mesh agent** —
its mesh identity and vendored skill kit are described under [Mesh identity](#mesh-identity-agentculture)
and [Skills](#skills) below.

## Commands

```bash
# First-time setup (creates .venv, installs the package editable with [server,tts])
./setup.sh

# Manager CLI (no robot deps needed)
reachy-mini-mcp overview          # status; also the no-arg default
reachy-mini-mcp show              # print the mcp.json snippet
reachy-mini-mcp install --client claude-code --scope project   # / uninstall
reachy-mini-mcp doctor            # diagnose deps, daemon, registration

# MCP server (stdio) — requires the daemon running first
./start.sh                        # wraps: source .venv/bin/activate && python -m reachy_mini_mcp serve
reachy-mini-mcp serve             # or: python -m reachy_mini_mcp serve
fastmcp run reachy_mini_mcp/server.py

# OpenAI-compatible HTTP server on :8100
./start_openai_server.sh
reachy-mini-mcp serve --openai    # or: python -m reachy_mini_mcp.server_openai

# Tests (manager CLI only — no robot stack/daemon needed). Avoid `uv sync`/`uv run
# pytest`: the universal resolve pulls the [server] extra → reachy-mini → pycairo,
# which needs system cairo to build. Install just the manager + pytest instead:
uv pip install -e . pytest pytest-xdist pytest-cov   # `uv venv` first if you have no venv
# Plain run, or with coverage (CI uses the --cov form; fail_under=95 gate applies):
uv run --no-project pytest -n auto -v
uv run --no-project pytest -n auto --cov=reachy_mini_mcp --cov-report=xml:coverage.xml --cov-report=term -v

# Piper TTS voice model download helper
./setup_piper_model.sh
```

Tests live in `tests/` and cover the manager CLI (mcp.json builder, client-config
merge/remove, command smoke tests) — they need no robot stack or daemon. The
robot runtime (`server.py`, `server_openai.py`, `tts_queue.py`, tool scripts) is
not unit-tested; verify it by running `serve` against a live daemon. The README
links `docs/conversation_stack.md`, `DOCKER_SETUP.md`, and `SEQUENCE_COMMANDS.md`,
which are not present in the repo.

## The tool repository (the core abstraction)

Tools are data + a script, not hardcoded Python. To add or change a robot operation you edit `reachy_mini_mcp/tools_repository/`, never the server files (it ships as package data in the wheel):

```text
reachy_mini_mcp/tools_repository/
├── tools_index.json     # registry: name → definition_file, with enabled flag
├── <tool>.json          # parameter schema + which script runs it
└── scripts/<tool>.py    # the actual logic
```

Loading flow (duplicated in both servers): `tools_index.json` → each `<tool>.json` → dynamically import `scripts/<tool>.py` via `importlib`. Every script must define:

```python
async def execute(make_request, create_head_pose, tts_queue, params):
    ...
    return {...}   # Dict[str, Any]
```

- `make_request(method, endpoint, json_data=, params=)` — the daemon HTTP helper. Movement goes through `POST /api/move/goto` with `{"head_pose": ..., "antennas": [...], "duration": ...}`; state via `GET /api/state/full`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentculture/reachy-mini-mcp](https://github.com/agentculture/reachy-mini-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->

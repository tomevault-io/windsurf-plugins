---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
# Install / sync dependencies
uv sync

# Lint
uv run ruff check src tests scripts
uv run mypy src

# Tests
uv run pytest tests/unit -v           # unit only (fast, no daemon)
uv run pytest tests/integration -v    # needs no running daemon; fixture spawns one
uv run pytest tests/ -v               # all

# Single test
uv run pytest tests/unit/test_envelope.py::test_request_roundtrip -v

# Regenerate docs/reference/WIRE_PROTOCOL.md after changing bus models
uv run python scripts/gen_protocol_doc.py

# Verify docs/reference/WIRE_PROTOCOL.md is in sync (used in CI equivalent)
uv run python scripts/gen_protocol_doc.py --check

# Reproduce the complete CI gate before every push
uv run ruff check .
uv run python scripts/check_brand.py
uv run python scripts/check_public_repo.py
uv run mypy src
uv run mypy --platform linux src
uv run pytest -q
uv run python scripts/gen_protocol_doc.py --check
uv build
uv run python scripts/smoke_wheel.py dist

# Run daemon manually
uv run coderook-core                        # foreground; Ctrl+C to stop
CODEROOK_PORT=8000 uv run coderook-core        # override port

# Send a ping
uv run coderook ping
uv run coderook --version
```

## Architecture

This is a **dual-process** local AI coding agent runtime. `coderook-core` is a persistent daemon that owns all state (sessions, runs, background workers, permissions, durable persistence); `coderook` and `coderook-tui` are thin clients that connect to it over loopback TCP.

```
coderook-core (daemon)
  ├─ 127.0.0.1:7437  TCP JSON-RPC 2.0 / NDJSON  (IPC for CLI & TUI, token auth)
  ├─ 127.0.0.1:7438  hand-written HTTP/1.1 + SSE (durable runtime API, Bearer auth)
  └─ spawns: fleet worker subprocesses, hooks, MCP servers, git/rg/pyright
       ↑
coderook (CLI/TUI)   coderook web (local browser SPA)
```

**TUI and Web are shared product frontends.** Both consume the same durable runtime events, receipts, permissions, sessions, and Change Center state. The scripted CLI exists for automation and debugging. `coderook` with no arguments launches the TUI; `coderook web` starts the local browser product; other subcommands form the scripted CLI surface.

**`docs/reference/FUNCTIONAL_ARCHITECTURE.md` is the authoritative architecture reference** (generated from a full code read; regenerate its claims against code when in doubt). The summary below is the orientation map.

### Protocol layer (`src/code_rook/core/bus/`)

All IPC messages are typed pydantic v2 models with a **discriminated union on the `type` field**. This is the contract boundary — adding a new command or event means adding a new model class to `commands.py` or `events.py` and extending the `Command`/`Event` union.

- `envelope.py` — `JsonRpcRequest`, `JsonRpcSuccess`, `JsonRpcError`, `EventPushEnvelope`, error code constants (`AUTH_REQUIRED=-32001`, `AUTH_FAILED=-32002`), `HandlerError`, `make_error()`
- `commands.py` — `Command` union across core/auth, headless run (`agent.run`), event subscribe/replay, durable `thread.*`/`turn.*`, `session.*`, permission/question responses, worker/workflow, workspace/turn inspection, and MCP/Hooks/Memory/background/artifact management
- `events.py` — `Event` union across run/step lifecycle, `agent.decision`, `tool.call_*`, `llm.*`, `context.*`, `permission.*`, `plan.*`, `subagent.*`, `background.*`, extensions, diagnostics and durable runtime events

`docs/reference/WIRE_PROTOCOL.md` is **generated** from these models by `scripts/gen_protocol_doc.py`. Always regenerate and commit it after changing bus models.

### Transport layer (`src/code_rook/core/transport/`)

- `socket_server.py` — TCP server (`asyncio.start_server`); reads NDJSON lines, dispatches each line as an independent task (so long handlers don't block concurrent commands like `permission.respond`), handles JSON-RPC error cases. Enforces loopback peer, first-frame `core.authenticate` with `hmac.compare_digest`. On `start()`, probes `host:port` first — errors if another daemon is already listening. Business handlers are registered in `app.py`.
- `socket_client.py` — shared client for CLI/TUI: token read, auth handshake, command/response futures, event dispatch.
- `ipc_broadcaster.py` — topic (fnmatch) + scope (`global`/`run:`/`thread:`) subscriptions; durable runtime replay with high-water handoff so reconnects lose no events.
- `auth.py` — IPC token lifecycle (`~/.coderook/ipc-token`, 0600, exclusive create, strict validation).

### HTTP runtime API (`src/code_rook/core/api/`)

Second interface on port 7438 for external integrations: hand-written HTTP/1.1 (no framework), Bearer auth, SSE event stream with `after_seq`/`Last-Event-ID` cursor replay. Endpoints under `/v1/` (threads/turns/events/interrupt/steer/items/receipt/capabilities/usage).

### Config (`src/code_rook/core/config.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyletser/coderook](https://github.com/kyletser/coderook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

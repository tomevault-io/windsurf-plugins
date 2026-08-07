---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Desktop app for **Grok (xAI)** in the style of Claude Code / Codex: chat, image/video
generation & editing, and an **agentic coding module** with local file access. It is a
**monorepo rebuild** of an older customtkinter (Python) app into **Electron (frontend) +
Python FastAPI sidecar (backend)**. The mature xAI logic (OAuth, SSE streaming, media) is
**reused, not rewritten**. Full plan & phase status live in [`docs/plans/REBUILD_PLAN.md`](docs/plans/REBUILD_PLAN.md)
(source of truth). The old customtkinter app has been **removed from the repo** (kept as an external
backup) — Phase 8 closed.

## The single most important structural fact

The shared xAI core lives at the **repo root**, NOT inside `caelo_core/`:
`config.py`, `api_manager.py`, `oauth_manager.py`, `chats_manager.py`, `history_manager.py`,
and `make_icon.py`. The `caelo_core` sidecar imports these as top-level modules (`import config`,
`from api_manager import APIManager`, …) — `caelo_core/__init__.py` prepends the repo root to
`sys.path` at import time so this works, and `caelo_core.spec` declares them as `hiddenimports`
with `pathex='.'`. (The now-removed `archive/app.py` also reused them via a `sys.path` shim — hence
they predate the sidecar — but the binding constraint is the sidecar + PyInstaller + data paths.)

**Do not move, rename, or restructure these root modules.** Doing so breaks `caelo_core` imports,
the PyInstaller build, and (via `config.py`) every data-file path. New backend code belongs in
`caelo_core/`; only touch the root modules to fix shared xAI logic.

## Architecture

```
Electron main (desktop/src/main/index.ts)
  • spawns the sidecar (dev: `python -m caelo_core`; packaged: resources/caelo-core/caelo-core.exe)
  • generates a session token → CAELO_CORE_TOKEN env; reads handshake line from sidecar stdout
  • /health monitor every 10s, auto-restart on crash (≤5 tries); kills sidecar on quit
        │  preload (contextBridge) exposes window.caelo  →  Renderer (React 19 + TS)
        ▼  HTTP REST + WebSocket — 127.0.0.1 only, bearer/query token
Python sidecar "caelo-core" (FastAPI/uvicorn, caelo_core/)
  • server.py mounts routers; state.py Backend wraps reused legacy managers + keys/settings
  • agent/ = coding-agent engine (workspace sandbox, permission gate, tools, llm, session loop)
        │  Bearer token (OAuth access token → API key → XAI_API_KEY) — only to api.x.ai
        ▼  xAI / Grok API
```

**Handshake:** the sidecar binds a free port on `127.0.0.1` and prints exactly one line to
stdout: `__CAELO_CORE_READY__ {"port":…,"token":…,"version":…}`. uvicorn logs go to stderr so
stdout stays clean. Electron parses this; the token it generated (passed via `CAELO_CORE_TOKEN`)
is authoritative. See [`caelo_core/__main__.py`](caelo_core/__main__.py) and `index.ts`.

**Auth precedence** (`Backend.get_api_key`/`_resolve_auth` in [`caelo_core/state.py`](caelo_core/state.py)):
the default (`auth_source="auto"`) is OAuth access token → saved `api_key` from settings → `XAI_API_KEY`
from `.env`. A **hard source switch** (`auth_source` in `caelo_settings.json`, set via Settings →
"Model source": auto/oauth/api_key) overrides this: `oauth` uses **only** the account token, `api_key`
uses **only** a key (settings → `.env`) — neither silently falls back to the other, so picking "API key"
with no key fails clearly instead of quietly using OAuth (the live A3 gap). `active_auth_source()` reports
the source actually in effect (`oauth|api_key|env|none`) — surfaced in `/auth/status` (+`has_stored_key`/
`has_env_key`); the renderer footer shows "Not signed in" when the sidecar is up but no source is active,
and the API-key field is removable (`DELETE /settings/api-key`) and masked (dots) when stored. OAuth uses the
public PKCE `client_id` of grok-cli/Hermes and undocumented `auth.x.ai` endpoints (see
[`config.py`](config.py)) — may break server-side without notice.

**Coding agent** ([`caelo_core/agent/`](caelo_core/agent/)): an event-driven LLM loop
(`session.py`) with file tools (`tools.py`: read_file/list_dir/glob/grep/write_file/edit_file/
run_command). READONLY tools run freely; MUTATING tools (write/edit/run) go through
`PermissionGate` (`permissions.py`) and require user approval unless "Always allow"-listed.
All file paths are sandboxed to the workspace root via `Workspace.resolve` (`workspace.py`,
rejects `..`/absolute escapes); `run_command` is NOT sandboxed in command content, hence approval.
The approval allowlist is persisted to `caelo_permissions.json` and shared by the agent WS and the
REST `/permissions` routes.

**Agent hardening (M1, see [`docs/plans/zrealizowane/PLAN_NAPRAWY.md`](docs/plans/zrealizowane/PLAN_NAPRAWY.md) P0-1…P0-8 — all done):**
`run_command` **rejects shell metacharacters** (`command_metachars` in `permissions.py`) so the
"Always allow" allowlist can't be bypassed by chaining (`git && rm`); the allowlist key is the full
normalized command, not the exe name. It runs with a **scrubbed env** (no `CAELO_CORE_TOKEN`/
`XAI_API_KEY`/secret-like vars), **tree-kills** on Stop/timeout (Windows `taskkill /T /F`), and Stop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AuraVixStudio/caelo](https://github.com/AuraVixStudio/caelo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**CountBot** is an open-source AI Agent framework and execution hub for Chinese users. It connects LLMs, IM channels, workflows, and external tools into a unified execution pipeline.

**Stack**: FastAPI (backend) + Vue 3 + TypeScript (frontend) + SQLite (database) + Python 3.10+
**Deployment**: Source (`python start_app.py`) or Desktop (PyInstaller-packaged, see releases)
**Ports**: Default 7000 (both `start_app.py` and `start_dev.py`). Configurable via `COUNTBOT_HOST` / `COUNTBOT_PORT`.

## Quick Commands

| Task | Command |
|------|---------|
| Start production | `python start_app.py` (default port: 7000) |
| Start dev (hot reload) | `python start_dev.py` (default port: 7000) |
| Start backend only | `uvicorn backend.app:app --reload --host 0.0.0.0 --port 7000` |
| Backend lint | `flake8 backend/` |
| Run tests | `python -m pytest tests/ -v` |
| Frontend dev | `cd frontend && npm run dev` |
| Frontend build | `cd frontend && npm run build` |
| Frontend lint | `cd frontend && npm run lint` |
| Frontend type-check | `cd frontend && npm run type-check` |
| Frontend unit tests | `cd frontend && npm run test` |
| Install backend deps | `pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/` |
| Create conda env | `conda env create -f environment.yml` |

**Default backend URL**: http://127.0.0.1:7000 (configurable via `COUNTBOT_HOST` / `COUNTBOT_PORT`)
**Frontend dev URL**: http://localhost:5173 (Vite proxies `/api/*` and `/ws` to backend at `http://127.0.0.1:8000` by default; set `COUNTBOT_PORT=8000` or update Vite proxy target to match)

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `COUNTBOT_HOST` | `127.0.0.1` | Bind address for the backend server |
| `COUNTBOT_PORT` | `7000` | Bind port for the backend server |

## Startup Flow

`backend/app.py` uses FastAPI `lifespan` context manager. On startup, in order:
1. Initialize DB (SQLAlchemy async engine)
2. Load config from `Setting` table via `ConfigLoader`
3. Resolve workspace path (via `WorkspaceManager`, falls back to `./workspace`)
4. Seed bundled workspace resources
5. Create `ProviderRuntimeState` with `KeyRotator` (round-robin + failover)
6. Create `ChannelManager` + **`await manager.async_init()`** (loads per-user channel configs from DB into memory, does NOT start them)
7. **`asyncio.create_task(channel_manager.start_all())`** — starts outbound dispatcher AND all channel supervision tasks on boot
8. Start `ChannelMessageHandler.start_processing()` (consumes inbound messages from the bus)
9. Initialize MCP client (if enabled, non-blocking background connection to stdio/SSE/streamable_http servers)
10. Initialize OSS uploader (optional, for image/media upload)
11. Initialize cron system — `CronExecutor` + `CronScheduler` + `HeartbeatService` + `ensure_heartbeat_job()`. Cron uses a separate `AgentLoop` (system-level, no user_id).
12. Mount WebSocket endpoint at `/ws/chat` — per-connection tool registry (session isolation), user auth via cookie or Bearer token. Local connections auto-assign first admin as default user. MCP tools synced to per-session registry on connect.
13. Mount frontend static files — SPA fallback for `/login` and `/setup/{secret}`.

**Remote setup secret**: If no password hash exists at startup, a one-time setup token is generated (TTL = `REMOTE_SETUP_SECRET_TTL_MINUTES`, default 30 min). Printed to console log as `/setup/{secret}`. Cleared automatically once password is set via the setup wizard.

**Graceful shutdown**: `channel_manager.stop_all()` → `cron_scheduler.stop()` → `atexit` fallback cleanup.

**Channel lifecycle**: On boot, `start_all()` starts supervision tasks for ALL enabled channels immediately. On user login, `start_user_channels(user_id)` is called but is a no-op for already-running channels. On logout, `stop_user_channels(user_id)` stops channels and cancels their supervision tasks (channels stay registered in the manager for re-start on next login).

## Backend Architecture

```
backend/
├── app.py                  # FastAPI entry + lifespan (config, DB, MCP, cron, channels)
├── database.py             # SQLAlchemy async engine + declarative Base + get_db dependency
│
├── api/                    # REST API routers (mounted at /api/*)
│   ├── chat.py             # Chat messages + WebSocket streaming
│   ├── auth.py             # Re-exports router from modules/auth/
│   ├── channels.py         # IM channel CRUD
│   ├── agent_teams.py      # Multi-agent team workflows
│   ├── cron.py             # Scheduled task management
│   ├── mcp.py / wiki.py    # MCP client & Wiki KB management
│   └── ...                 # settings, tools, personalities, skills, memory, tasks, queue, system
│
├── modules/
│   ├── agent/              # Agent core
│   │   ├── loop.py         # AgentLoop: ReAct cycle (LLM → tool → observe → repeat)
│   │   ├── workflow.py     # WorkflowEngine: pipeline / graph / council modes
│   │   ├── context.py      # ContextBuilder: assembles system prompt + history + memory + skills

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shen-An/mybot](https://github.com/Shen-An/mybot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

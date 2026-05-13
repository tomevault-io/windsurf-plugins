---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Claude DevFleet — Autonomous Coding Agent Platform

## What It Is
Multi-agent coding platform that dispatches Claude Code agents to work on missions (coding tasks). Each agent runs in an isolated git worktree with MCP-powered tools, can create sub-missions for other agents, and coordinates through a dependency-aware dispatch system. Includes a status page / health monitoring system.

## Architecture
```
frontend/          → React 19 + Vite UI (port 3101 via nginx in Docker, port 3100 local dev)
backend/           → FastAPI + SQLite (port 18801, uvicorn --reload)
```

Runs in Docker via `docker-compose.yml` or locally with Python venv.

### Dispatch Engine (Dual-Mode)
Controlled by `DEVFLEET_ENGINE` env var:

- **`sdk` (default)** — Uses `claude-code-sdk` Python API via `sdk_engine.py`. Attaches stdio MCP servers for context + tools. Structured message streaming.
- **`cli` (legacy)** — Uses `dispatcher.py` to spawn `claude` CLI as subprocesses and parse `stream-json` stdout. Set `DEVFLEET_ENGINE=cli` to use.

### Background Services
Started in `app.py` lifespan:
- **Mission Watcher** (`mission_watcher.py`) — Polls every 5s for `auto_dispatch=1` missions whose dependencies are met, dispatches to available agent slots
- **Scheduler** (`scheduler.py`) — Checks cron schedules every 60s, clones template missions with `auto_dispatch=1`
- **Health Checker** (`health_checker.py`) — Polls monitored services for uptime

## Build & Run Commands

```bash
# Local dev (recommended)
python3 -m venv venv && source venv/bin/activate
pip install -r backend/requirements.txt
cd backend && uvicorn app:app --host 0.0.0.0 --port 18801 --reload
cd frontend && npm run dev    # port 3100

# Docker
docker compose up -d
docker compose build devfleet-ui && docker compose up -d devfleet-ui
docker logs devfleet-api -f

# Check for running agents before restarting
docker top devfleet-api | grep claude
```

There are no tests or linting configured in this project.

## Key Files
- `backend/app.py` — FastAPI routes: projects, missions, dispatch, resume, remote-control, sessions, reports, dashboard, auto-loop, scheduling, system status, MCP configs, services, health checks, incidents
- `backend/sdk_engine.py` — SDK dispatch engine: claude-code-sdk streaming, stdio MCP server attachment, report file pickup, per-project MCP config loading, cost tracking
- `backend/mission_watcher.py` — Auto-dispatch engine: polls for eligible missions, checks `depends_on` via `json_each`, dispatches to available slots, emits mission_events
- `backend/scheduler.py` — Cron scheduler: built-in cron parser, clones template missions on schedule, sets auto_dispatch
- `backend/mcp_context.py` — Stdio MCP server: contextual intelligence (mission, project, session, team context)
- `backend/mcp_devfleet.py` — Stdio MCP server: agent self-service (submit_report, create_sub_mission with auto_dispatch, request_review, get_sub_mission_status, list_project_missions)
- `backend/mcp_external.py` — MCP server: external integration endpoint (plan, dispatch, cancel, wait, dashboard — Streamable HTTP at /mcp, SSE legacy at /mcp/sse)
- `backend/planner.py` — AI project planner: natural language → project + chained missions via Claude Sonnet
- `backend/plugins.py` — Plugin system: auto-loads plugins from plugins/ dir, registers custom MCP tools + lifecycle hooks
- `backend/autoloop.py` — Auto-loop: parallel-aware planner (returns single or multiple tasks), multi-mission dispatch, waits for all to complete
- `backend/dispatcher.py` — CLI dispatch engine (legacy): builds CLI args, spawns `claude` CLI, parses stream-json
- `backend/remote_control.py` — Remote control manager: spawns `claude remote-control`, parses URL, monitors sessions
- `backend/db.py` — SQLite schema + auto-migrations (aiosqlite)
- `backend/models.py` — Pydantic models: DispatchOptions, MissionCreate/Update (with parent_mission_id, depends_on, auto_dispatch, schedule_cron), McpServerCreate
- `backend/prompt_template.py` — Builds full prompt from mission + last report
- `backend/worktree.py` — Git worktree isolation for agents

## MCP Servers (auto-attached to every agent)
Two stdio MCP servers spawned as subprocesses per agent dispatch:

**devfleet-context** — Context intelligence:
- `get_mission_context`, `get_project_context`, `get_session_history`, `get_team_context`, `read_past_reports`

**devfleet-tools** — Agent self-service:
- `submit_report` — Writes JSON to `data/reports/{session_id}.json`, picked up by sdk_engine after completion
- `create_sub_mission` — Creates mission with `auto_dispatch=true`, `parent_mission_id` set. Supports `wait_for_me` for dependency control
- `request_review` — Creates review mission with `depends_on=[current_mission]` for auto-dispatch after completion
- `get_sub_mission_status` — Check progress of child missions
- `list_project_missions` — Browse project missions

Tool naming in allowed_tools: `mcp__devfleet-context__get_mission_context`, `mcp__devfleet-tools__submit_report`, etc.

**context-mode** (optional) — Context savings + session continuity:
- Enabled per-dispatch via `context_mode: true` in DispatchOptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LEC-AI/claude-devfleet](https://github.com/LEC-AI/claude-devfleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

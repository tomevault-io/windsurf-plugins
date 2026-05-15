---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Minions is an autonomous task management system with a Kanban board UI. Users create tasks via a chat interface; each task is a Hermes agent session that autonomously decides how to execute — doing the work itself, spawning child sessions, or creating cron jobs. A periodic heartbeat checks in on in-progress tasks, prompting them to self-report status. The user never talks to child sessions or cron jobs directly; the task agent manages its own sub-resources.

## Prerequisites

- Node.js v18+
- Hermes agent installed with its Python venv (default location: `~/.hermes/hermes-agent/`)

The server spawns a Python worker subprocess that imports Hermes `AIAgent` directly — no `hermes gateway` process or HTTP API is involved. The Python executable is resolved in this order:

1. `HERMES_PYTHON` env var (explicit path)
2. `HERMES_AGENT_DIR/venv/bin/python` (if `HERMES_AGENT_DIR` is set)
3. `~/.hermes/hermes-agent/venv/bin/python` (default)
4. `python3` (system fallback)

## Commands

```bash
npm run dev          # dev mode: tsx watch + Vite dev server on :6969
npm run build        # production build: server (tsc) + client (vite) + copy .sql/.py assets
npm run start        # run compiled production build
npm run prod         # build + run production in one command
```

No test suite or linter is configured.

## Architecture

```
Browser (React/Vite :6969)
  ↕ HTTP + SSE
Express API + Vite middleware (:6969)
  ↕ JSONL over stdin/stdout
Python worker (hermes_worker.py)
  ↕ direct Python import
Hermes AIAgent
```

- **Server** (`server/`): Express + SQLite (better-sqlite3, WAL mode). All timestamps are epoch milliseconds.
- **Python worker** (`server/workers/hermes_worker.py`): JSONL bridge that imports Hermes `AIAgent` directly. Spawned as a subprocess by `HermesWorkerAdapter`, auto-restarts on crash. Manages concurrent agent runs via semaphore (default: 10).
- **Client** (`client/`): React 19 + Vite + Tailwind CSS + Zustand + react-router. `@shared` path alias resolves to `../shared/`.
- **Shared** (`shared/types.ts`): TypeScript types used by both client and server.

### State directory

All persistent state lives under `MINIONS_HOME` (default: `~/.minions/`):
- `data/minions.db` — SQLite database
- `logs/` — log files
- `workspace/` — default working directory for Hermes task artifacts

## Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Agent communication | Python subprocess + JSONL | Imports Hermes AIAgent directly — no HTTP gateway overhead, structured streaming events, per-task model/reasoning control |
| Task execution | Autonomous agent session | Each task IS a Hermes session. The agent decides execution strategy (self, child session, cron job). Our backend doesn't manage sub-resources. |
| Heartbeat | DB-configured scheduler in Express (15 min default) | Sends check-in to task sessions. Agent reports status honestly; system updates task accordingly. |
| Source of truth | Hermes SessionDB for chat history; Minions SQLite for task metadata; in-memory LiveChatRun for active streams | Hermes owns all transcripts and replay. Minions has no message table. `tasks.id` is the Hermes root session ID; Minions stores task metadata, per-task settings, heartbeat logs, and `last_agent_response_at` for heartbeat idle gating. During active streaming, `live-chat.ts` holds an in-memory `LiveChatRun` with accumulated messages. After streaming ends and the run TTL expires, chat history is projected from Hermes SessionDB on demand. |
| Status ownership | AI moves to `blocked`/`in_review` via heartbeat; human moves everything else via drag-drop | Clean separation: AI reports status, human controls all manual transitions. |

## Key Patterns

- **Session lifecycle**: `tasks.id` is the Minions task ID and the Hermes root session ID. Chat, history, metadata reads, and heartbeat check-ins all use `task.id`; Minions does not persist Hermes-returned child or continuation session IDs.
- **Chat projection**: `GET /tasks/:id/messages` loads raw rows from Hermes `SessionDB.get_messages()` via the Python worker, which filters out heartbeat check-ins (`[AUTOMATED CHECK-IN]`), status blocks (`<status_report>`), and tool-call-only turns. The client shows optimistic messages during streaming and loads the projected history from Hermes on page load/task switch.
- **Per-task model/reasoning**: Each task can override the default Hermes model and reasoning effort (`agent_model`, `reasoning_effort` columns on `tasks`). Settings logic lives in `server/agent-settings.ts`. The Python worker resolves the final model/provider from Hermes config + per-task overrides.
- **Heartbeat status parsing**: Agent responses must contain a `<status_report>` / `</status_report>` block with JSON. Unparseable responses are logged but don't crash.
- **Heartbeat visibility**: `summary` and optional `user_summary` are saved to `heartbeat_log` and shown only in the Activity tab. Heartbeat turns remain in Hermes session history for agent continuity, but the chat projection filters them out.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agent-3-7/hermes-agent-mission-control](https://github.com/Agent-3-7/hermes-agent-mission-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->

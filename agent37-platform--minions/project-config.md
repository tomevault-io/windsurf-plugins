---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Engineering practices

We're a startup. You're probably used to writing enterprise code — code that tries to handle every possible edge case and has fallbacks for everything. That's not how we do things around here: our number one rule is to keep things simple. We handle ONLY the most important cases.

We try to only add new functionality that is small (that is, simple and few lines of code) or absolutely necessary. If a change is not small or absolutely necessary, don't make it.

## What This Is

Minions is an autonomous task management system with a Kanban board UI. Users create tasks via a chat interface; each task is a Hermes agent session that autonomously decides how to execute — doing the work itself, spawning child sessions, or creating Hermes cron jobs shown in Minions as Scheduled Tasks. Successful agent runs move tasks to review automatically. The user never talks to child sessions directly; recurring work is managed from the Scheduled Tasks page.

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
- `skills/` — installed skills, registered with Hermes via `external_dirs`

## Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Agent communication | Python subprocess + JSONL | Imports Hermes AIAgent directly — no HTTP gateway overhead, structured streaming events, per-task model/reasoning control |
| Task execution | Autonomous agent session | Each task IS a Hermes session. The agent decides execution strategy (self, child session, Hermes cron job/scheduled task). Our backend doesn't manage child sessions. |
| Review transition | Successful agent runs move to review | After a chat or goal run completes successfully, the server records the response metadata and moves an `in_progress` task to `in_review` in the same update. No separate completion evaluation runs outside the conversation. |
| Source of truth | Hermes SessionDB for chat history; Minions SQLite for task metadata; in-memory LiveChatRun for active streams | Hermes owns all transcripts and replay. Minions has no message table. `tasks.id` is the Hermes root session ID; Minions stores task metadata, per-task settings, and `last_agent_response_at`. During active streaming, `live-chat.ts` holds an in-memory `LiveChatRun` with accumulated messages. After streaming ends and the run TTL expires, chat history is projected from Hermes SessionDB on demand. |
| Status ownership | Successful runs auto-move to `in_review`; human moves everything else via drag-drop | Clean separation: the system queues completed agent work for review, and humans control final completion. |

## Key Patterns

- **Session lifecycle**: `tasks.id` is the Minions task ID and the Hermes root session ID. Chat and history reads all use `task.id`; Minions does not persist Hermes-returned child or continuation session IDs.
- **Chat projection**: `GET /tasks/:id/messages` loads raw rows from Hermes `SessionDB.get_messages()` via the Python worker, which filters out tool-call-only turns and empty messages. The client shows optimistic messages during streaming and loads the projected history from Hermes on page load/task switch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent37-platform/minions](https://github.com/agent37-platform/minions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->

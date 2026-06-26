---
trigger: always_on
description: - Repo: Open-source monorepo ([omnigentx/jarvis](https://github.com/omnigentx/jarvis))
---

# Jarvis AI Assistant — Repository Guidelines

- Repo: Open-source monorepo ([omnigentx/jarvis](https://github.com/omnigentx/jarvis))
- Core runtime: [fast-agent](https://fast-agent.ai/) (Python, git submodule at `backend/fast-agent`)
- Always reference files repo-root relative (e.g. `backend/routes/chat.py:187`); never absolute paths.

## Core Principles

1. **Realtime monitoring** — SSE, WebSocket, or equivalent push protocol. **Absolutely no polling.** All agent status, tool execution progress, and activity events must stream to clients in real-time via `services/activity_stream.py` (ActivityStreamManager) and `services/sse_progress.py` (ProgressManager).

2. **Production-ready code** — Every feature must handle edge cases: connection drops, auth failures, SSE reconnection with exponential backoff, timeout handling, graceful degradation. No TODO placeholders or "happy path only" implementations.

3. **Clarification before implementation** — When requirements are ambiguous, always present specific questions with options to the user before writing code. Prefer numbered questions that are actionable.

4. **Clean architecture** — Code must be modular, well-separated, and easy to extend:
   - Keep files under ~500 LOC; split when exceeding.
   - Extract reusable composables/services; avoid copy-paste.
   - Use clear naming that reflects domain concepts (not generic names).
   - Every component should have a single responsibility.

5. **fast-agent best practices** — This project is built on [fast-agent](https://fast-agent.ai/). Always:
   - Reference official docs before implementing agent features: [Tool Runner](https://fast-agent.ai/agents/tool_runner/), [Prompting](https://fast-agent.ai/agents/prompting/), [Instructions](https://fast-agent.ai/agents/instructions/)
   - Use `ToolRunnerHooks` for monitoring and progress tracking (see `services/spawn_progress_bridge.py` for the pattern).
   - Persist runtime-created agents through `services/agent_definitions.py` (SQLite). Replaces the legacy file-based agent cards.
   - Understand the session/history model via `services/session_service.py`.
   - Never bypass fast-agent's built-in capabilities; extend through hooks, not monkey-patching.

6. **Evidence-based debugging** — Never guess or assume root causes. Always:
   - Read source code of the relevant module before concluding.
   - Provide file paths and line numbers when identifying issues.
   - Show logs/stack traces as evidence.
   - If the bug is in a dependency (e.g. `fast-agent` core), read the submodule source at `backend/fast-agent/`.

## Project Structure

```
jarvis/
├── backend/                    # FastAPI + fast-agent runtime (Python)
│   ├── server.py               # FastAPI application entry point
│   ├── agent.py                # fast-agent agent definitions (@fast.agent decorators)
│   ├── routes/                 # API endpoints
│   │   ├── agents.py           # /api/agents — list, detail, pause/resume, team, skills, context
│   │   ├── chat.py             # /api/chat (legacy single-shot, used by Xiaozhi), /api/chat-stream (SSE)
│   │   ├── inject.py           # /api/agents/{name}/inject — prompt injection (MessageBus + resume)
│   │   ├── agent_timeline.py   # /api/agents/{name}/timeline (SSE)
│   │   ├── approvals.py        # /api/approvals — human-in-the-loop approval system
│   │   ├── scheduler.py        # /api/scheduler — cron jobs CRUD + SSE stream
│   │   ├── notifications.py    # /api/notifications — push notification management
│   │   ├── tts.py              # /api/tts/* — text-to-speech streaming
│   │   ├── stories.py          # /api/stories/* — audiobook reader + crawl
│   │   ├── library.py          # /api/library — book library management
│   │   ├── sessions.py         # /api/sessions — conversation history
│   │   ├── token_usage.py      # /api/tokens — LLM token usage metrics
│   │   └── auth.py             # /api/auth — login, API key check
│   ├── services/               # Business logic & shared state
│   │   ├── shared_state.py     # Singleton refs (agent_app, spawn_bridge, registry_db, etc.)
│   │   ├── session_service.py  # Session management (resume_and_send)
│   │   ├── sse_progress.py     # SSE progress manager + create_progress_hooks()
│   │   ├── activity_stream.py  # ActivityStreamManager (realtime agent status SSE)
│   │   ├── spawn_progress_bridge.py  # Subprocess events → SSE bridge + team completion
│   │   ├── spawn_event_socket.py     # Unix socket server for MCP subprocess events
│   │   ├── inject_resume.py    # Resume non-running agents with context from DB
│   │   ├── context_persistence.py    # Save/load agent context windows to/from SQLite
│   │   ├── pause_manager.py    # Agent pause/resume state management
│   │   ├── approval_service.py # Human-in-the-loop approval workflow
│   │   ├── cron_scheduler.py   # Cron job execution engine (APScheduler)
│   │   ├── background_jobs.py  # Background task management
│   │   ├── meeting_events.py   # Meeting event stream manager
│   │   ├── meeting_hooks_bridge.py   # Meeting hooks → SSE bridge
│   │   ├── dynamic_agents.py   # Dynamic agent loading from SQLite (services/agent_definitions.py)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omnigentx/jarvis](https://github.com/omnigentx/jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->

---
trigger: always_on
description: This file is written for AI coding agents (Claude Code, Cursor, Copilot, etc.) working on the Analytics Agent codebase. Read it before making changes.
---

# AGENTS.md — Analytics Agent Codebase Guide

This file is written for AI coding agents (Claude Code, Cursor, Copilot, etc.) working on the Analytics Agent codebase. Read it before making changes.

---

## Project in one sentence

Analytics Agent is a LangGraph-based chat agent that uses **DataHub** tools for metadata context and pluggable **SQL engines** (Snowflake first) to answer natural-language data questions, with Vega-Lite charts rendered inline in a React + Vite UI served by the same FastAPI process.

---

## Running the stack

A `justfile` at the repo root covers all common tasks. Install `just` once (`brew install just`), then:

```bash
just install          # uv sync + pnpm install
just start            # build frontend if stale, start backend at :8100
just port=8102 start  # same on a custom port
just stop             # kill the backend
just nuke             # wipe the DB (start from scratch / re-trigger wizard)
just start-remote     # start + print DataHub connection status
just logs             # tail /tmp/analytics_agent.log
just test             # unit tests
just build            # force frontend rebuild
```

`just start` automatically detects whether `frontend/src` is newer than `frontend/dist` and rebuilds only when needed.

### Without just (manual)

```bash
uv sync
cd frontend && pnpm install && pnpm build && cd ..
uv run uvicorn analytics_agent.main:app --reload --port 8101
# → http://localhost:8101
# The setup wizard handles LLM key + connections on first run.
# Optional: cp .env.example .env to pre-configure credentials.
```

### Two-process mode (frontend hot reload)

```bash
# Terminal 1 — backend (dev)
uv run uvicorn analytics_agent.main:app --reload --port 8101

# Terminal 2 — Vite dev server with HMR
cd frontend && pnpm dev
# → http://localhost:5173 (proxies /api/* to :8101)
```

**DataHub credentials**: run `datahub init --sso --host https://your-instance.acryl.io/gms` once. The app reads `~/.datahubenv` automatically; or set `DATAHUB_GMS_URL` + `DATAHUB_GMS_TOKEN` in `config.yaml` / `.env`.

**Database**: SQLite at `./data/dev.db` by default. Alembic runs automatically on startup. For Postgres set `DATABASE_URL=postgresql+asyncpg://...`.

---

## Key file map

| Path | What it does |
|------|-------------|
| `backend/src/analytics_agent/main.py` | FastAPI app factory + lifespan (runs Alembic, seeds integrations, mounts SPA) |
| `backend/src/analytics_agent/agent/graph.py` | LangGraph `StateGraph`: ReAct agent → conditional chart node |
| `backend/src/analytics_agent/agent/streaming.py` | `astream_events` → SSE event dicts; handles `on_tool_error` |
| `backend/src/analytics_agent/agent/history.py` | Reconstructs LangChain message history from DB rows; pads orphaned tool calls |
| `backend/src/analytics_agent/agent/chart_tool.py` | `create_chart` LangChain tool; stores spec in `_pending_charts` side-channel |
| `backend/src/analytics_agent/agent/chart_generator.py` | `chart_node`: runs after SQL results; calls chart LLM → updates `pending_chart` state |
| `backend/src/analytics_agent/api/chat.py` | `POST /api/conversations/{id}/messages` → `StreamingResponse` (SSE) |
| `backend/src/analytics_agent/api/settings.py` | Connection CRUD + test + tool toggles + prompt + display settings |
| `backend/src/analytics_agent/api/oauth.py` | SSO browser flow, PAT storage, OAuth popup flow, credential encryption |
| `backend/src/analytics_agent/context/datahub.py` | Builds DataHub LangChain tools via `datahub_agent_context.build_langchain_tools()` |
| `backend/src/analytics_agent/engines/resolver.py` | **Single credential resolution point** — loads Integration + credential from DB |
| `backend/src/analytics_agent/engines/snowflake/engine.py` | Snowflake `QueryEngine`: execute_sql, list_tables, get_schema, preview_table; SSO/key-pair/PAT auth |
| `backend/src/analytics_agent/engines/factory.py` | Engine registry; `register_engine` / `unregister_engine` for dynamic connections |
| `backend/src/analytics_agent/db/models.py` | SQLAlchemy models: Conversation, Message, Integration, IntegrationCredential, Setting |
| `backend/src/analytics_agent/db/repository.py` | Repos: ConversationRepo, MessageRepo, SettingsRepo, IntegrationRepo, CredentialRepo |
| `backend/src/analytics_agent/prompts/system_prompt.md` | Agent system prompt (edit here — loaded at runtime) |
| `frontend/src/components/Chat/ChatView.tsx` | Chat shell; handles welcome-screen → new conversation flow |
| `frontend/src/components/Chat/WelcomeView.tsx` | Landing screen with LLM greeting, suggestion chips, engine selector |
| `frontend/src/components/Settings/SnowflakeAuthSection.tsx` | Segmented auth selector: Password / Private Key / SSO / PAT / OAuth |
| `frontend/src/store/conversations.ts` | Zustand: conversations, messages, engines, streaming state |
| `frontend/src/store/display.ts` | Zustand: app name, logo, cached LLM greeting |

---

## Integrations + credential architecture

Connections are stored in two DB tables:

- **`integrations`** — connection topology (account, warehouse, database, user). `source="yaml"` for `config.yaml` connections, `source="ui"` for UI-created ones.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datahub-project/analytics-agent](https://github.com/datahub-project/analytics-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->

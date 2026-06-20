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

A `Makefile` at the repo root covers all common tasks (`make` is available everywhere — no install needed):

```bash
make install          # uv sync + pnpm install
make start            # build frontend if stale, start backend at :8100
make PORT=8102 start  # same on a custom port
make stop             # kill the backend
make nuke             # wipe the DB (start from scratch / re-trigger wizard)
make start-remote     # start + print DataHub connection status
make logs             # tail /tmp/analytics_agent.log
make test             # unit tests
make build            # force frontend rebuild
```

`make start` uses Make's native dependency tracking: it rebuilds the frontend only when `frontend/src` files are newer than `frontend/dist/index.html`.

### Without make (manual)

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

**Database**: SQLite at `./data/dev.db` by default. For Postgres set `DATABASE_URL=postgresql+asyncpg://...`.

**Bootstrap (migrations + seed)**: The FastAPI lifespan no longer runs migrations or seeds — it does read-only initialization (loading engines from the DB, propagating env vars, validating the encryption key). All DB-mutating bootstrap work lives in `analytics_agent.bootstrap` and is invoked via the CLI: `analytics-agent bootstrap` runs Alembic migrate → seed-integrations → seed-context-platforms → seed-defaults, idempotently. Run it before the first `uvicorn` start and after any release that adds migrations. The Helm chart runs it automatically as a `pre-install`/`pre-upgrade` hook.

---

## Key file map

| Path | What it does |
|------|-------------|
| `backend/src/analytics_agent/main.py` | FastAPI app factory + lifespan (read-only init: loads engines, validates encryption key, mounts SPA — no migrations/seeds) |
| `backend/src/analytics_agent/agent/graph.py` | LangGraph `StateGraph`: ReAct agent → conditional chart node |
| `backend/src/analytics_agent/agent/streaming.py` | `astream_events` → SSE event dicts; handles `on_tool_error` |
| `backend/src/analytics_agent/agent/history.py` | Reconstructs LangChain message history from DB rows; pads orphaned tool calls |
| `backend/src/analytics_agent/agent/chart_tool.py` | `create_chart` LangChain tool; stores spec in `_pending_charts` side-channel |
| `backend/src/analytics_agent/agent/chart_generator.py` | `chart_node`: runs after SQL results; calls chart LLM → updates `pending_chart` state |
| `backend/src/analytics_agent/api/chat.py` | `POST /api/conversations/{id}/messages` → `StreamingResponse` (SSE) |
| `backend/src/analytics_agent/api/settings.py` | Connection CRUD + test + tool toggles + prompt + display settings |
| `backend/src/analytics_agent/api/oauth.py` | SSO browser flow, PAT storage, OAuth popup flow, credential encryption |
| `backend/src/analytics_agent/context/datahub.py` | Builds DataHub LangChain tools via `datahub_agent_context.build_langchain_tools()` |
| `backend/src/analytics_agent/engines/factory.py` | Engine registry + `ConnectorSpec` map; native connectors (Snowflake, BigQuery) resolved to MCP subprocesses |
| `backend/src/analytics_agent/engines/mcp/engine.py` | `MCPQueryEngine` — discovers tools from a subprocess via `get_tools_async()`, caches client to keep subprocess alive |
| `backend/src/analytics_agent/engines/sqlalchemy/engine.py` | In-process engine for MySQL, PostgreSQL, SQLite via SQLAlchemy |
| `backend/src/analytics_agent/api/connectors.py` | Connector lifecycle: `GET /api/connectors/{type}/status`, `POST /api/connectors/{type}/install`, `POST /api/connectors/{type}/test` |
| `connectors/snowflake/` | Standalone MCP server package — runs as a subprocess launched by the core via `uvx`; owns all Snowflake deps |
| `connectors/bigquery/` | Standalone MCP server package — same pattern for BigQuery/GCP deps |
| `backend/src/analytics_agent/db/models.py` | SQLAlchemy models: Conversation, Message, Integration, IntegrationCredential, Setting |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datahub-project/analytics-agent](https://github.com/datahub-project/analytics-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->

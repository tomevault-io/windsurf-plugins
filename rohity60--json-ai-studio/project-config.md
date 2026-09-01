---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

JSON AI Studio — AI-powered JSON configuration management. Developers/PMs manage JSON configs via natural language chat. OpenAPI-first MVP scaffold.

## Quickstart

```bash
# API (Python 3.12+, FastAPI, uv)
cd apps/api && uv sync && uv run uvicorn src.json_ai_studio.main:app --reload
# → http://localhost:8000, Swagger at /docs

# Web (Next.js 15 app router, React 19, Tailwind 3, TypeScript)
cd apps/web && npm install && npm run dev
# → http://localhost:3000
```

No tests, no linting, no CI. Run `pytest` or `ruff` manually if installed.
Python files: always format with `black .` after editing (install via `uv add --dev black`).

## Structure

```
json-ai-studio/
├── openapi/spec.yaml            ← Single source of truth. 26 endpoints, 33 schemas. OpenAPI 3.1 YAML.
├── apps/api/                    ← FastAPI backend, layered per ADR-0013
│     └── src/json_ai_studio/
│           ├── main.py              ← Thin app factory: FastAPI init, lifespan (DB engine), CORS, include_router per controller.
│           ├── controllers/         ← APIRouters (health, sessions, versions, uploads, chat, diffs, explain, users). Map domain errors → HTTP codes.
│           ├── services/            ← Business logic (session, version, chat SSE, diff, user provisioning, credit quota). Only layer touching stores. errors.py = domain exceptions.
│           ├── db/session_store.py  ← SessionStore ABC + InMemorySessionStore singleton (ADR-0013, amends ADR-0006). Async interface, DB-ready seam.
│           ├── db/database.py       ← Async SQLAlchemy engine + session factory (ADR-0015). None when DATABASE_URL unset → anonymous-only mode.
│           ├── db/models_orm.py     ← SQLAlchemy ORM models (users table). Schema changes go through Alembic.
│           ├── models.py            ← Pydantic schemas (canonical wire format per ADR-0011).
│           ├── settings.py          ← pydantic-settings config: Auth0, DATABASE_URL, quota limits, LOG_* knobs (env vars / .env).
│           ├── logging_config.py    ← Env-driven logging (ADR-0017). Console + size-rotating file handler; captures uvicorn access logs. configure() called at import in main.py.
│           ├── auth.py              ← Principal resolution: anonymous X-API-Key or Auth0 bearer (ADR-0015) + sliding-window rate limiter.
│           ├── auth0_jwt.py         ← PyJWT + PyJWKClient RS256 token verification (JWKS cached 1h).
│           └── utils.py             ← deepdiff-based diff engine + business-rule validation (V-04: timeout > 0, retryCount <= 10).
│     └── alembic/                 ← Alembic migrations (async env.py). Run: uv run alembic upgrade head.
├── apps/web/                    ← Next.js 15 app router frontend
│     ├── src/
│     │    ├── app/
│     │    │    ├── layout.tsx   ← Root layout. Wraps children in <SessionProvider>.
│     │    │    └── page.tsx     ← Home. Two-panel layout with header, chat/upload tabs, diff preview toggle.
│     │    ├── components/
│     │    │    ├── UploadPanel.tsx  ← Drag-and-drop + textarea for JSON upload.
│     │    │    ├── ChatPanel.tsx    ← SSE chat consumer (thinking → diff(s) → complete phases).
│     │    │    ├── JSONTree.tsx     ← react-json-view-lite wrapper for JSON preview.
│     │    │    ├── DiffViewer.tsx   ← Before/after side-by-side + per-change accept/reject.
│     │    │    └── VersionSidebar.tsx ← Version snapshot list + select toggle.
│     │    ├── context/
│     │    │    └── SessionContext.tsx  ← React context: session lifecycle, SSE chat, diff actions, export. Persists sessionId to localStorage + mirrors workspace to IndexedDB (ADR-0014).
│     │    └── lib/
│     │         ├── api.ts            ← Minimal fetch-based API client (no retry, no interceptor).
│     │         └── versionCache.ts   ← IndexedDB workspace mirror (idb pkg). Versions + working JSON survive backend restarts; restored via POST .../versions/restore.
│     └── tailwind.config.ts   ← Tailwind config.
├── docs/adr/                  ← 15 Architecture Decision Records. All tech decisions numbered & dated.
├── docker-compose.yml         ← api + web + db (postgres:16, host port 5433).
└── openapi/README.md           ← OpenAPI spec conventions.
```

## Key facts

- **OpenAPI-first contract**: `openapi/spec.yaml` (26 endpoints, 33 schemas) is the contract source. Pydantic models in `models.py` implement it verbatim (ADR-0011). Frontend TS types are hand-written mirrors — not auto-generated.
- **Session model**: In-memory store behind async `SessionStore` ABC (`db/session_store.py`, ADR-0013). Each session = working_json + version snapshots + conversation history. No TTL. Browser mirrors versions + working JSON to IndexedDB and bulk-restores into a fresh session after backend restart (ADR-0014).
- **Chat flow**: User NL message → LiteLLM call → field-level diffs → SSE stream: thinking → diff(s) → complete (ADR-0002, ADR-0004). The system prompt in `main.py` embeds the current working JSON as few-shot examples (ADR-0005).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rohity60/json-ai-studio](https://github.com/rohity60/json-ai-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->

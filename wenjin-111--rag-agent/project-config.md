---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

Argus is a RAG (Retrieval-Augmented Generation) knowledge base platform with:
- **Backend**: Python FastAPI + SQLAlchemy async + PostgreSQL/pgvector
- **Frontend**: Vue 3 + TypeScript + Vite + Element Plus
- **Infrastructure**: Docker Compose (PostgreSQL, MinIO, Elasticsearch)
- **Admin console**: platform-wide document/group/QA-history management, insights, system health, audit logs

## Build Commands

```bash
# Backend (Python)
cd Argus-python
pip install -r requirements.txt
python init_db.py              # Initialize DB tables + seed admin
uvicorn app.main:app --host 0.0.0.0 --port 10001 --reload

# Frontend (Vue 3)
cd Argus-frontend
npm install
npm run dev                    # Dev server on http://localhost:5173

# Infrastructure
docker compose up -d           # Start PG + MinIO + ES
docker compose ps              # Check health
docker compose down            # Stop (data preserved in volumes)
```

## Tech Stack

- **Python 3.12+** with FastAPI, SQLAlchemy 2.0 async, asyncpg
- **Vue 3.5** (Composition API, `<script setup>`), TypeScript, Vite 8
- **PostgreSQL 16 + pgvector** — HNSW vector index, cosine distance
- **Elasticsearch 8.x** — BM25 keyword search, IK Chinese tokenizer
- **MinIO** — S3-compatible object storage for documents
- **LangChain / LangGraph** — AI agent framework (ReactAgent + MemorySaver)
- **Pydantic v2** — Settings and request validation
- **JWT** (PyJWT) — Access token auth
- **Passlib + bcrypt** — Password hashing
- **Axios** — Frontend HTTP client (with snake_case→camelCase interceptor)
- **KaTeX + DOMPurify** — Markdown/LaTeX rendering (frontend, unified in `utils/markdown.ts`)

## Architecture

```
Argus-python/app/
├── main.py                        # FastAPI entry, lifespan, routers, periodic cleanup task
├── config.py                      # Pydantic Settings (env_file=.env)
├── dependencies.py                # Async engine + session factory
├── database.py                    # SQLAlchemy Base
├── common/                        # Shared: ApiResponse, exceptions, middleware, UserContext
├── auth/                          # JWT auth, login/register/refresh, password hashing
├── user/                          # Account settings, admin user CRUD + create user
├── group/                         # Group CRUD, memberships, invitations, join requests + admin router
├── document/                      # Upload (direct + chunked), list, preview, download, delete + admin router
│   └── maintenance.py             # Expired upload-session cleanup (hourly)
├── ingestion/                     # ETL pipeline: parse → clean → chunk → vectorize → ES index
├── qa/                            # RAG QA: query planning → hybrid retrieval → LLM generation
│   ├── models.py                  #   QaSession / QaMessage (persisted, with evidence_level)
│   ├── history_service.py         #   QA history queries (user + admin views)
│   ├── retrieval.py               #   Parallel vector+ES, RRF fusion, evidence assessment
│   └── service.py                 #   QA orchestration, streaming SSE, persistence
├── assistant/                     # AI Agent: ReactAgent + tool calling + session memory
│   ├── agent/                     #   LangGraph agent factory, KB search tool
│   └── memory/                    #   Short-term memory manager (LLM semantic summary)
├── metrics/                       # LLM usage tracking, stats, platform insights
├── models_config/                 # Admin model config management (chat + embedding)
├── audit/                         # Audit trail (sensitive operations) + admin router
├── system/                        # System health check endpoint
└── engine/                        # Infrastructure adapters
    ├── vector_store.py            #   PGvector adapter (custom SQL, LangChain-free)
    ├── es_service.py              #   Elasticsearch index + search
    └── storage.py                 #   MinIO client
```

## Key Conventions

### Database
- All DB access via SQLAlchemy 2.0 async (`select()`, `update()`, `delete()`)
- Use `async_session_factory` from `dependencies.py` for manual sessions
- Route handlers get sessions via `Depends(get_db)` (auto-commit on success, rollback on exception)
- Models use `Mapped[]` type annotations, extend `Base` from `database.py`
- New tables auto-created on startup (`Base.metadata.create_all`); one-off column additions
  use idempotent `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` in `main.py:_init_database`

### API Response
- Every controller returns `ApiResponse` — `success`, `data`, `message`
- Throw `BusinessException`(400) / `ForbiddenException`(403) / `AuthenticationException`(401)
- Snake_case in Python dicts → camelCase via frontend Axios interceptor
- **Exceptions**: `qa/ask`, `qa/stream-ask` (direct payload), `assistant/sessions` list/detail/context (direct payload)
- **Pagination convention**: admin list endpoints return `{items, total, page, limit}`

### Auth Flow
- JWT Bearer token in `Authorization` header → `JwtAuthenticationFilter` dependency
- `get_current_user` → returns `AuthenticatedUser` record (also sets `UserContext` — read in `LoggingMiddleware` for `userId=` in access logs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wenjin-111/RAG_agent](https://github.com/Wenjin-111/RAG_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

---
trigger: always_on
description: This file provides guidance for AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working with code in this repository.

## Project Overview

Poco is a multi-service AI agent execution platform that orchestrates Claude AI agents to perform coding tasks. The system consists of four main components:

- **Frontend** (Next.js 16) - Web UI for task management and monitoring
- **Backend** (FastAPI) - API server, database management, and session orchestration
- **Executor** (FastAPI + Claude Agent SDK) - Agent execution engine with hook-based extensibility
- **Executor Manager** (FastAPI + APScheduler) - Task scheduling and dispatch service

## Architecture Flow

1. User creates task via Frontend
2. Executor Manager receives task, creates session via Backend
3. Executor Manager schedules task with APScheduler
4. Task Dispatcher sends task to Executor with callback URL
5. Executor runs Claude Agent SDK with configured hooks
6. Hooks send progress callbacks to Executor Manager during execution
7. Executor Manager forwards callbacks to Backend for persistence
8. Frontend polls Backend for session status updates

## Development Commands

### Frontend (Next.js)

```bash
cd frontend
pnpm install        # Install dependencies
pnpm dev            # Development server
pnpm build          # Build for production
pnpm start          # Start production server
pnpm lint           # ESLint
pnpm format         # Prettier
```

### Python Services (Backend, Executor, Executor Manager)

Each Python service has its own directory with a `pyproject.toml`. Run commands from within the service directory:

```bash
cd <service>        # backend/, executor/, or executor_manager/
uv sync             # Install dependencies
uv run python -m app.main    # Run development server
# Or directly with uvicorn:
uvicorn app.main:app --reload
```

### Database Migrations (Backend)

```bash
cd backend
uv run -m alembic revision --autogenerate -m "description"  # Autogenerate migration (review and adjust)
uv run -m alembic upgrade head                               # Apply migrations
uv run -m alembic downgrade -1                               # Rollback one migration
```

Guideline: Always start migrations with `--autogenerate` + `upgrade head`, then manually review and adjust the generated revision file. Do not hand-write a migration from scratch as the first step.

### Pre-commit Hooks

```bash
pre-commit install      # Install hooks
pre-commit run --all-files  # Run manually
```

## Technology Stack

**Frontend:** Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS v4, shadcn/ui, pnpm

**Backend Services:** Python 3.12+, FastAPI, Uvicorn, SQLAlchemy 2.0, Alembic, Pydantic Settings, PostgreSQL

**Executor:** claude-agent-sdk, FastAPI

**Executor Manager:** APScheduler, httpx, FastAPI

**Package Managers:** UV for Python, pnpm for Node.js

**Python Package Index:** Tsinghua mirror (<https://pypi.tuna.tsinghua.edu.cn/simple>)

## Code Organization

### Backend (`backend/app/`)

- `api/v1/` - API endpoints (sessions, callback)
- `core/` - Settings, error handlers, middleware, observability
- `models/` - SQLAlchemy models (agent_session, agent_message, tool_execution, usage_log)
- `repositories/` - Data access layer (session_repository, message_repository, etc.)
- `schemas/` - Pydantic schemas (session, callback, response)
- `services/` - Business logic (session_service, callback_service)
- `main.py` - FastAPI app factory

### Executor (`executor/app/`)

- `core/` - AgentExecutor engine, workspace management, callback client
- `hooks/` - Hook system for extensibility (base, manager, callback, todo, workspace)
- `utils/` - Serializer, git platform clients (GitHub, GitLab)
- `schemas/` - Request, response, callback, state schemas and enums
- `api/v1/` - Task execution callback endpoint

### Executor Manager (`executor_manager/app/`)

- `core/settings.py` - Service configuration
- `scheduler/` - APScheduler config and task dispatcher
- `services/` - Backend and executor API clients
- `schemas/` - Task and callback schemas
- `api/v1/` - Task creation, status, and callback endpoints

### Frontend (`frontend/`)

- `app/` - App Router routes, layouts, loading states, proxy routes
- `features/` - Domain modules (chat, projects, capabilities, scheduled-tasks, etc.)
- `components/` - Cross-feature shared components (`ui/`, `shared/`, `shell/`)
- `hooks/` - Cross-feature reusable hooks only
- `lib/` - Framework-agnostic utilities, i18n setup, startup preload logic
- `services/` - Global infrastructure only (e.g., API client); do not put feature business logic here
- `types/` - Global shared types only; feature-specific types stay in each feature

## Key Design Patterns

- **Repository Pattern** - Data access abstraction in `backend/app/repositories/`
- **Service Layer** - Business logic in `backend/app/services/`
- **Hook Pattern** - Plugin-based extensibility in `executor/app/hooks/`
- **Abstract Base Classes** - Git platform clients extend `BaseGitClient`

## Environment Configuration

Each Python service requires a `.env` file. See `backend/.env.example` for the Backend template.

**Backend:** DATABASE_URL, HOST, PORT, CORS_ORIGINS, SECRET_KEY, DEBUG
**Executor Manager:** backend_url, callback_base_url, max_concurrent_tasks, callback_token

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [poco-ai/poco-claw](https://github.com/poco-ai/poco-claw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

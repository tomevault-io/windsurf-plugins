---
trigger: always_on
description: Data Talks is a full-stack AI-powered data analysis platform. Users connect data sources (CSV, XLSX, SQL databases, BigQuery, Google Sheets, GitHub, dbt) and ask questions in natural language. The platform generates SQL/Python code, executes it, and returns answers with charts.
---

# CLAUDE.md — Data Talks

## Project Overview

Data Talks is a full-stack AI-powered data analysis platform. Users connect data sources (CSV, XLSX, SQL databases, BigQuery, Google Sheets, GitHub, dbt) and ask questions in natural language. The platform generates SQL/Python code, executes it, and returns answers with charts.

## Tech Stack

- **Frontend**: React 18 + TypeScript, Vite 5, Tailwind CSS, shadcn/ui (Radix), React Query v5, React Router v6
- **Backend**: Python 3.11+, FastAPI 0.115+, SQLAlchemy 2.0+ (async), Alembic, Pydantic v2
- **Database**: SQLite (default) or PostgreSQL (via DATABASE_URL)
- **LLM**: OpenAI / Ollama / LiteLLM (configurable per user)
- **Auth**: JWT (python-jose + passlib/bcrypt), optional login mode

## Quick Reference Commands

```bash
# Install everything
make install

# Development (backend + frontend with hot reload)
make dev

# Build frontend and run production-like
make run

# Database migrations
make migrate

# Frontend only
npm run dev          # Dev server on :8080
npm run build        # Production build
npm run lint         # ESLint
npm test             # Vitest
npm run typecheck    # tsc --noEmit

# Backend only
cd backend && uv run data-talks run   # Starts on :8000
cd backend && uv run alembic upgrade head
cd backend && pytest
```

## Project Structure

```
├── src/                  # React frontend
│   ├── components/       # Reusable components
│   │   └── ui/           # shadcn/ui primitives (do NOT edit manually)
│   ├── pages/            # Route-level page components
│   ├── services/         # API client and service functions
│   ├── hooks/            # Custom React hooks
│   ├── contexts/         # React context providers
│   ├── lib/              # Shared utilities
│   └── utils/            # Helper functions
├── backend/
│   └── app/
│       ├── main.py       # FastAPI app entry point
│       ├── models.py     # SQLAlchemy models (all tables)
│       ├── schemas.py    # Pydantic request/response schemas
│       ├── database.py   # DB engine and session setup
│       ├── auth.py       # JWT auth utilities
│       ├── config.py     # Settings from environment
│       ├── routers/      # API route handlers
│       ├── scripts/      # Per-source-type Q&A and summary scripts
│       ├── llm/          # LLM client abstraction and utilities
│       └── services/     # Business logic (email, webhooks, alerts)
├── Makefile              # Common dev commands
├── vite.config.ts        # Vite bundler config
├── tailwind.config.ts    # Tailwind theme and plugins
└── package.json          # Frontend dependencies
```

## Key Conventions

### Language
- **All code, comments, commit messages, and docs MUST be in English.**
- UI strings for end users go through i18n (`LanguageContext`) and support PT/EN/ES.

### Frontend
- Components use PascalCase filenames (e.g., `SourcesPanel.tsx`).
- Hooks use camelCase with `use` prefix (e.g., `useAuth.ts`).
- Use `api()` from `src/services/apiClient.ts` for all backend requests.
- Use `useAuth()` for authentication state.
- Use `useLanguage()` for translated strings.
- Path alias: `@/` maps to `src/`.
- shadcn/ui components in `src/components/ui/` — do not edit directly; use `npx shadcn-ui@latest add <component>` to add new ones.
- React Query for server state; avoid manual fetch + useState patterns.

### Backend
- Endpoints that touch tenant-scoped models (Source, Agent, PipelineRun, …) use `Depends(require_membership)` to resolve a `TenantScope` (user + organization_id + role), then filter every query with `tenant_filter(Model, scope)`.
- Write/delete endpoints add `Depends(require_role("member"))` / `Depends(require_role("admin"))`.
- User-personal endpoints (LlmConfig, QA sessions, dashboards) still use `Depends(require_user)`.
- Database sessions injected via `Depends(get_db)`.
- Source-specific Q&A logic lives in `app/scripts/ask_<type>.py`.
- LLM calls go through `app/llm/client.py` (never call OpenAI SDK directly).
- All DB operations are async (use `await` with SQLAlchemy async session).
- Secrets in `Source.metadata_` and Telegram/WhatsApp/Slack bot tokens are Fernet-encrypted at rest; see `app/services/crypto.py`.
- New tables/columns require an Alembic migration.

### Database
- IDs are UUID v4 strings (not auto-increment integers).
- Guest mode uses fixed `GUEST_USER_ID`; admin uses `ADMIN_USER_ID`. The guest user is auto-enrolled as `owner` of a single `Guest` organization.
- Multi-tenancy: `Organization` + `OrganizationMembership` drive the active-tenant scope. A User may belong to N Orgs; the active one travels in the JWT `org_id` claim or is bound to the ApiKey row.
- Role hierarchy (ascending): `viewer < member < admin < owner`. `require_role("admin")` lets admins + owners through.

### Git & PRs
- Keep commits focused and descriptive.
- PR descriptions must include a summary and test plan.
- Do not push directly to `main`; use feature branches.

## Environment Variables

Backend config lives in `backend/.env`. Key variables:
- `LLM_PROVIDER`: `openai` | `ollama` | `litellm`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Empreiteiro/data-talks](https://github.com/Empreiteiro/data-talks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

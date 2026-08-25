---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

### Backend (`cd backend`)

```bash
uv sync --all-extras --dev          # install deps
uv run uvicorn app.main:app --reload  # dev server (localhost:8000)

# Quality checks
uv run ruff format --check app tests
uv run ruff check app tests
uv run mypy app tests
uv run pytest                        # all unit tests
uv run pytest tests/integration      # integration tests (needs Postgres)
uv run pytest tests/drive/test_router.py::test_list_items_returns_page  # single test
uv run pytest -k "test_quota"        # filter by name
uv run pytest --cov=app --cov-report=term-missing
```

Integration tests require a running PostgreSQL instance. The default URL is `postgresql+asyncpg://postgres:postgres@localhost:5432/clouddrive_test` (set via `DATABASE_URL` env var or start with `docker compose up postgres`).

### Frontend (`cd frontend`)

```bash
npm ci                    # install deps
npm run dev               # dev server (localhost:5173)
npm run lint
npm run typecheck
npm run test -- --run     # all unit tests (one-shot)
npm run test              # watch mode
npx vitest run src/api/authApi.test.ts   # single file
npx vitest run -t "stores access token"  # filter by test name
npm run build
npm run playwright:install  # first-time only
npm run test:e2e
```

### Docker

```bash
docker compose up --build   # start all services
docker compose up postgres  # Postgres only (for integration tests)
```

## Architecture

### Backend

Every domain is a self-contained package under `backend/app/<module>/` with four files: `router.py`, `service.py`, `repository.py`, `schemas.py`. Modules never import from each other's internals — they import each other's services via FastAPI dependency injection.

```
app/
  core/          config, security (JWT), exceptions, dependencies
  models/        SQLAlchemy ORM models (all imported into Base metadata)
  schemas/       shared Pydantic response types (DriveItemResponse, Page, etc.)
  api/v1/router.py   aggregates all module routers
  <module>/
    router.py    FastAPI routes; depends on service via _module_service factory
    service.py   business logic; takes AsyncSession + other services
    repository.py  DB queries (raw SQLAlchemy)
    schemas.py   module-specific Pydantic I/O schemas
```

**Auth flow:** `get_current_user_id` (in `core/dependencies.py`) extracts a UUID from the JWT Bearer token. Every protected router depends on `CurrentUserId = Annotated[UUID, Depends(get_current_user_id)]`.

**Error handling:** Raise typed subclasses of `AppError` — `NotFoundError` (404), `ForbiddenError` (403), `QuotaExceededError` (413), `NameConflictError` (409), `InvalidOperationError` (422). The global handler in `main.py` converts them to `{ "error": { "code": "...", "message": "...", "details": {} } }`.

**Migrations:** `backend/alembic/` — run `uv run alembic upgrade head` to apply.

**Storage:** `StorageProvider` protocol (in `app/storage/`) abstracts file I/O. `LocalStorageProvider` is the only implementation. The `LOCAL_STORAGE_PATH` env var must be set before app import because `get_settings()` is `@lru_cache`.

**Data model notes:**
- `user_item_preferences` table stores per-user starred state (not a column on `drive_items`)
- "Recent" items are derived from `activity_logs`, not `drive_items.updated_at`
- Refresh token and public share token are stored as hashes only

### Frontend

```
src/
  api/           authApi, driveApi, uploadApi, etc. — thin wrappers over axios
  app/           AuthInitializer, RequireAuth, RedirectIfAuth, router, ProtectedLayout
  stores/        authStore (Zustand) — access token only; uiStore, uploadStore
  hooks/         useAuth, useDrive, useUpload, etc. — TanStack Query wrappers
  pages/         one file per route
  components/    drive/, layout/, preview/, share/, trash/, upload/
```

**Auth / token lifecycle:**
- `authStore` holds the access token in memory only — no localStorage, no sessionStorage, no Zustand persist middleware.
- On every page load, `AuthInitializer` (`src/app/AuthInitializer.tsx`) calls `POST /auth/refresh` before the router renders. If the HttpOnly refresh token cookie is still valid, the access token is restored in Zustand and the user stays on their page. Until the attempt settles, `AuthInitializer` returns `null` to prevent `RequireAuth` from prematurely redirecting.
- `RequireAuth` then just reads `authStore.accessToken` — non-null means authenticated.
- The Axios interceptor in `client.ts` handles 401s on subsequent API calls by calling refresh again and retrying. It uses a separate `refreshClient` (no interceptors) to avoid infinite loops.

**API client:** `src/api/client.ts` exports `api` (main Axios instance) and `refreshClient` (interceptor-free, for refresh calls). `toApiError()` reads `d['code']` directly from the response body — MSW error handlers must return `{ code: '...', message: '...' }` flat, not nested.

**State:** Server state via TanStack Query (v5); UI/upload/auth state via Zustand (v5). Query keys are defined per-hook file using factory functions (`driveKeys.items(parentId)` etc.).

## Testing Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [billwu101/CloudDrive](https://github.com/billwu101/CloudDrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

BugFuse is a lightweight, self-hosted error tracking platform (a Sentry-compatible
alternative). It ingests events from Sentry SDKs via the Sentry envelope/store
protocols, groups them into issues, and surfaces them through a web UI with alerting
automations. It is a monorepo: `backend/` (FastAPI + async SQLAlchemy + Postgres),
`frontend/` (React 19 + Vite + Tailwind v4), and `website/` (public site: Astro +
Starlight docs + Tailwind v4).

## Commands

### Backend (run from `backend/`, uses `uv`)
- Install deps: `uv sync`
- Run dev server (autoreload, 127.0.0.1:8000): `uv run dev` (defined as a project script; `uv run start` runs without reload)
- Lint: `uv run ruff check`. **Do NOT run `uv run ruff format`** — the pinned ruff (0.15.6) has a formatter bug that strips the parentheses from multi-type `except (A, B):` clauses, producing invalid Python 3 (this is the original source of the `except A, B:` errors that were fixed). Use `ruff check --fix` for import sorting; format by hand until ruff is repinned.
- Tests: `uv run pytest` (from `backend/`). The suite builds a real Postgres schema from Alembic migrations; by default it uses **Testcontainers** (needs Docker), or set `TEST_DATABASE_URL=postgresql+asyncpg://…` to point at an existing Postgres. Each test runs in a rolled-back transaction. See `backend/tests/README.md`. The app targets Python **3.14** (it uses `uuid.uuid7` and PEP 649 deferred annotations); the suite also runs on 3.13 via small shims in `tests/conftest.py`.
- Type check: the installed dev dependency is **mypy** (`uv run mypy app`), but the pre-commit *pre-push* hook invokes Astral's `ty check app tests`. `ty` is **not** in `uv.lock`/deps — that hook relies on it being available externally. Confirm which checker is intended before relying on either.
- DB migrations: `uv run alembic upgrade head` — Create a migration: `uv run alembic revision --autogenerate -m "message"`
- API docs served at `/api/docs` (Swagger) and `/api/redoc` once running.

### Frontend (run from `frontend/`, uses `pnpm`)
- Install: `pnpm install`
- Dev server (port 3000, proxies `/api` → `localhost:8000`): `pnpm dev`
- Build (runs typecheck first): `pnpm build`
- Combined gate: `pnpm check` (= `pnpm typecheck && pnpm lint`)
- Format: `pnpm format` (Prettier)

### Website (run from `website/`, uses `pnpm`)
- Public site (landing + docs) at bugfuse.com: Astro with Starlight docs at `/docs`.
- Install: `pnpm install` — Dev server: `pnpm dev` — Build: `pnpm build`
- Landing page: `src/pages/index.astro`; docs are markdown in `src/content/docs/docs/`.

### Full stack
- `docker compose up` brings up Postgres, runs migrations (`migrate` service), then backend and frontend. App is exposed on `BUGFUSE_PORT` (default 3000).

### Conventions enforced by pre-commit / CI
- Commits **must** be Conventional Commits, `--strict`, with scope from `{backend, frontend, website, infra, ci}` and type from `{build, chore, ci, docs, feat, fix, refactor, test}`. Example: `feat(backend): add release tracking`.
- `ruff check`/`ruff format` run on commit; `ty check` runs on push. Install hooks with `pre-commit install` (it installs `pre-commit`, `pre-push`, and `commit-msg` stages).
- Python target is **3.14**; ruff line-length 100 (E501 ignored), double quotes.

## Backend architecture

### Module layout convention
Each domain under `backend/app/<domain>/` follows a consistent split:
`models.py` (SQLAlchemy ORM), `schemas.py` (Pydantic I/O), `router.py` (FastAPI
endpoints), `service.py` (business logic), `queries.py` (read queries),
`dependencies.py` (FastAPI `Depends` guards). Routers are wired together in
`app/main.py`. Stick to this layering when adding features — keep DB/business
logic out of routers.

**Target layering.** Every HTTP/CRUD domain has been migrated onto a single
convention so business logic is unit-testable without HTTP (`projects`,
`organizations`, `invites`, `auth`, `auth/tokens`, `notification_channels`,
`views`, `events`, `issues` + `activities`, `environments`, `automations`
rules/executions, `sentry/artifacts`, `sentry/ingest` DSN auth). The
`sentry/ingest` request-scoped DSN credential check (`get_sentry_project`) is
migrated — it uses `TransactionalSession`, a `queries.py` lookup, and domain
errors — but the actual ingest hand-off after auth is still fire-and-forget.
The remaining fire-and-forget paths (`event_processing`, the automation
engine/dispatcher, post-ingest symbolication, and the bearer-token auth lookup)
intentionally keep managing their own sessions/transactions and still use the
legacy `DatabaseSession`. The convention:
- `router.py` — HTTP only: dependency wiring, request/response schemas, status
  codes. No SQLAlchemy queries, no `db.begin()`, no `HTTPException`.
- `service.py` — business logic: `(db, *, typed args)` in, ORM/DTO out, raises
  **domain errors** (`app/shared/errors.py`: `DomainError` →
  `ValidationError`/`NotFoundError`/`ConflictError`/`PermissionDeniedError`/
  `GoneError`). Never raises `HTTPException`. Does not own session lifecycle.
- `queries.py` — non-trivial read queries (session in, data out).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hemantapkh/bugfuse](https://github.com/hemantapkh/bugfuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

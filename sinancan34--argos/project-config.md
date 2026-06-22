---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Argos is a Pixel Code Audit Tool — a Chrome extension that automates browser actions and validates tracking pixel fires against user-defined rules. It has a FastAPI backend for scenario persistence and a Chrome extension frontend.

- **Backend:** FastAPI + SQLAlchemy + Alembic, Python 3.12
- **Extension:** WXT (Manifest V3) + React 19 + TypeScript + Tailwind CSS v4 + shadcn/ui

### Prerequisites

- Python 3.12+
- Node.js 18+
- Google Chrome

## Commands

### Backend

All commands run from `backend/` with the virtual environment activated (`source env/bin/activate`).

```bash
pip install -r requirements.txt        # Install dependencies
python -m uvicorn app.main:app --reload # Run dev server (http://127.0.0.1:8000)
alembic upgrade head                    # Run migrations
alembic revision --autogenerate -m "description"  # Generate migration
alembic downgrade -1                    # Rollback one migration
pytest                                  # Run all tests
pytest tests/test_scenarios_api.py      # Run a single test file
pytest tests/test_scenarios_api.py::test_create_scenario -v  # Run a single test
pytest --cov=app --cov-report=term-missing  # Run tests with coverage
```

Copy `.env.example` to `.env` before first run. Tests use an in-memory SQLite database (see `tests/conftest.py` for fixtures). No linter or formatter is configured.

### Extension

All commands run from `extension/`.

```bash
npm install          # Install dependencies
npm run dev          # Dev mode (opens Chrome with extension loaded)
npm run build        # Production build → extension/.output/chrome-mv3/
npm run test         # Run all tests (vitest)
npm run test:watch   # Run tests in watch mode
npm run test:coverage # Run tests with coverage report
```

No lint or typecheck scripts are configured. Tests use Vitest.

### Running Together

Both the backend and extension must run simultaneously. Start the backend dev server first, then `npm run dev` in the extension directory. The extension connects to `http://127.0.0.1:8000` by default (configurable via `VITE_API_BASE_URL` at build time).

## Shared Definitions

The `shared/` directory contains JSON files that are the single source of truth for both backend and extension:

- `shared/commands.json` — Command definitions (goto, click), selector strategies, match types
- `shared/validations.json` — Field constraints for scenarios, parameters, and selectors; enum values for sortBy, sortOrder
- `shared/providers.json` — Tracking provider definitions (e.g., Google Analytics) with URL patterns and parameter suggestions for autocomplete

Both sides load from these files: backend via `command_registry.py` / `validation_registry.py`, extension via `lib/commands.ts` / `lib/validation-registry.ts`. When adding or modifying commands, parameters, or validation rules, update the shared JSON first — both sides derive from it.

**Validation registry pattern:** `validations.json` defines field constraints (`type`, `required`, `minLength`, `maxLength`, `min`, `max`, `positive`, `minItems`, `conditionalRequired`). The backend converts these to Pydantic `Field()` kwargs via `pydantic_field_kwargs()`. The extension converts them to Zod schemas via `buildStringSchema()`, `buildIntSchema()`, `buildEnumSchema()`. Enum fields reference other JSON sources (e.g., `"source": "commands.matchTypes"`). Conditional validation (e.g., `value` required only when `match` is not `exists`) uses `@model_validator(mode="after")` in Pydantic and `.refine()` in Zod.

## Backend Architecture

**Domain model:** `Scenario` → has many `TestRun` → has many `TestRunResult`. All PKs are UUID strings generated in Python. Timestamps are ISO 8601 strings (not SQL datetime), generated via `datetime.now(timezone.utc).isoformat()`. JSON columns store `steps`, `validations` (on Scenario) and `actual_value` (on TestRunResult).

**Response envelope:** Single resource: `{ data: {...} }`. List resource: `{ data: [...], meta: { page, size, total_count, total_pages }, links: { self, first, last, next, prev } }`. Delete resource: `{ data: { id: "..." } }`. Error: `{ error: { code, message, details[] } }`.

**API versioning:** All routes are under `/api/v1/` prefix. Health check is at `/health` (root level).

**Key patterns:**
- Database sessions via FastAPI dependency injection: `db: Session = Depends(get_db)`
- All ORM models inherit from `Base` (in `database.py`) and live in `app/models.py`
- Pydantic schemas (request/response) and enums (MatchType, SelectorStrategy, SortBy, SortOrder) live in `app/schemas.py`, separate from ORM models
- New models must be imported in `alembic/env.py` for autogenerate to detect them
- Business logic lives directly in routers (no services layer)
- Config uses Pydantic BaseSettings with `.env` file loading
- SQLite by default (`sqlite:///./argos.db`), configurable via `DATABASE_URL`
- `connect_args={"check_same_thread": False}` is set for SQLite compatibility — must be adjusted if switching to PostgreSQL
- CORS origins configurable via `CORS_ALLOWED_ORIGINS` env var (comma-separated, defaults to `*`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sinancan34/argos](https://github.com/sinancan34/argos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->

---
trigger: always_on
description: - Run `/pre-commit` to execute the full pre-commit checklist for this project.
---

# CLAUDE.md

## Claude Code

- Run `/pre-commit` to execute the full pre-commit checklist for this project.

## Overview

REST API for managing football players built with Python and FastAPI. Implements
async CRUD operations with SQLAlchemy 2.0 (async), SQLite, Pydantic validation,
and in-memory caching.

## Tech Stack

- **Language**: Python 3.13
- **Framework**: FastAPI + Uvicorn
- **ORM**: SQLAlchemy 2.0 (async) + aiosqlite
- **Database**: SQLite (local/test), PostgreSQL-compatible
- **Migrations**: Alembic (async, `render_as_batch=True`)
- **Validation**: Pydantic
- **Caching**: aiocache (in-memory, 10-minute TTL)
- **Testing**: pytest + pytest-cov + httpx
- **Linting/Formatting**: Flake8 + Black
- **Containerization**: Docker

## Structure

```text
main.py         — application entry point: FastAPI setup, router registration
alembic.ini     — Alembic configuration (sqlalchemy.url set dynamically)
alembic/        — Alembic migration environment and version scripts
routes/         — HTTP route definitions, caching + dependency injection [HTTP layer]
services/       — async business logic                                   [business layer]
schemas/        — SQLAlchemy ORM models (database schema)                [data layer]
databases/      — async SQLAlchemy session setup + get_database_url()
models/         — Pydantic models for request/response validation
scripts/        — shell scripts for Docker (entrypoint.sh, healthcheck.sh)
tools/          — legacy standalone seed scripts (superseded by Alembic migrations)
rest/           — HTTP request file (players.rest) for manual API testing
gunicorn.conf.py — production WSGI worker config (used by Docker entrypoint)
tests/          — pytest integration tests
```

**Layer rule**: `Routes → Services → SQLAlchemy → SQLite`. Routes handle HTTP
concerns only; business logic belongs in services. Never skip a layer.

## Coding Guidelines

- **Naming**: snake_case (files, functions, variables), PascalCase (classes)
- **Type hints**: Required everywhere — functions, variables, return types
- **Async**: All routes and service functions must be `async def`; use
  `AsyncSession` (never `Session`); use `aiosqlite` (never `sqlite3`); use
  SQLAlchemy 2.0 `select()` (never `session.query()`)
- **API contract**: camelCase JSON via Pydantic `alias_generator=to_camel`;
  Python internals stay snake_case
- **Models**: `PlayerRequestModel` (no `id`, used for POST/PUT) and
  `PlayerResponseModel` (includes `id: UUID`, used for GET/POST responses).
  One request model intentionally covers both POST and PUT — per-operation
  differences (conflict check on POST, mismatch guard on PUT) are handled at
  the route layer, not by duplicating the model. Never reintroduce the removed
  `PlayerModel`; it was removed because a single flat model conflated ORM,
  request, and response concerns.
- **Primary key**: UUID surrogate key (`id`) — opaque, internal, used for GET
  by id only. UUID v4 for API-created records; UUID v5 (deterministic) for
  migration-seeded records. `squad_number` is the natural key —
  human-readable, domain-meaningful, used for all mutation endpoints (PUT,
  DELETE) and preferred for all external consumers
- **Caching**: cache key `"players"` (hardcoded); clear on POST/PUT/DELETE;
  `X-Cache` header (HIT/MISS)
- **Errors**: Catch specific exceptions with rollback in services; Pydantic
  validation returns 422 (not 400); squad number mismatch on PUT returns 400
  (not 422 — it is a semantic error, not a validation failure)
- **Logging**: `logging` module only; never `print()`
- **Line length**: 88; complexity ≤ 10
- **Import order**: stdlib → third-party → local
- **Tests**: integration tests against the real SQLite DB (seeded via
  Alembic migrations) via `TestClient` — no mocking. Naming pattern
  `test_request_{method}_{resource}_{context}_response_{outcome}`;
  docstrings single-line, concise; `tests/player_fake.py` for test data;
  `tests/conftest.py` provides a `function`-scoped `client` fixture for
  isolation; `tests/test_main.py` excluded from Black;
  `tests/test_migrations.py` covers Alembic downgrade paths
- **Decisions**: justify every decision on its own technical merits; never use
  "another project does it this way" as a reason — that explains nothing and
  may mean replicating a mistake
- **Avoid**: sync DB access, mixing sync/async, `print()`, missing type hints,
  unhandled exceptions

## Commands

### Quick Start

```bash
# Setup (using uv)
uv venv
source .venv/bin/activate  # Linux/macOS; use .venv\Scripts\activate on Windows
uv pip install --group dev

# Apply migrations (required once before first run, and after down -v)
uv run alembic upgrade head

# Run application
uv run uvicorn main:app --reload --port 9000       # http://localhost:9000/docs

# Run tests
uv run pytest                                      # run tests
uv run pytest --cov=./ --cov-report=term           # with coverage (target >=80%)

# Linting and formatting
uv run flake8 .
uv run black --check .

# Migration workflow
uv run alembic upgrade head                        # apply all pending migrations
uv run alembic downgrade -1                        # roll back last migration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanotaboada/python-samples-fastapi-restful](https://github.com/nanotaboada/python-samples-fastapi-restful) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

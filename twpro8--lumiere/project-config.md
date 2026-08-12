---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Codex CLI, Cursor, OpenCode, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Codex CLI, Cursor, OpenCode, etc.) when working with code in this repository.

## Project overview

Lumiere is a real-time chat application: a FastAPI backend (Python 3.14) and a React 19 frontend (TanStack Router/Query), backed by Postgres and Redis, run together via Docker Compose.

## Commands

### Environment setup

```bash
cp .env.example .env               # first-time setup, from repo root
docker compose watch               # start full stack (backend, frontend, db, redis, mailcatcher, adminer, traefik)
```

Local URLs: frontend `:5173`, backend `:8000` (docs at `/docs`), Adminer `:8080`, Mailcatcher `:1080`, Traefik UI `:8090`.

You can stop an individual Compose service and run it natively while the rest of the stack keeps working (same ports), e.g. `docker compose stop backend` then run the backend commands below directly.

### Backend (`backend/`, managed with `uv`)

```bash
cd backend
uv run uvicorn src.main:app --reload         # run dev server natively

bash scripts/lint.sh                          # mypy (strict) + ruff check + ruff format --check
bash scripts/format.sh                        # ruff check --fix + ruff format
bash scripts/test.sh                          # prestart (migrations) + pytest + coverage report/html

uv run pytest tests/                          # run tests directly
uv run pytest tests/integration/chats -k test_name   # single test/module
uv run pytest tests/unit                      # unit tests only

uv run alembic revision --autogenerate -m "message"  # new migration
uv run alembic upgrade head                          # apply migrations
```

Tests require `ENVIRONMENT=testing` and a running Postgres + Redis (see `.env.test`, `pytest.ini` loads `../.env.test` then `../.env`). `tests/conftest.py` overrides the DB session and Redis client dependencies, drops/recreates all tables, and seeds data (`tests/seeder.py`) once per session. Tests are split into `tests/unit/`, `tests/integration/<module>/`, `tests/e2e/`.

### Frontend (`frontend/`, managed with `pnpm`)

```bash
pnpm install
pnpm run dev            # Vite dev server, proxies /api -> localhost:8000
pnpm run build           # tsc -b && vite build
pnpm run lint            # oxlint
pnpm run format          # prettier --write (also sorts imports)
pnpm run format:check
pnpm run test            # vitest run
pnpm run test:watch      # vitest watch
```

From the repo root, `pnpm run dev` / `pnpm run lint` / `pnpm run generate-client` proxy to the frontend workspace.

### Cross-cutting

```bash
bash scripts/generate-client.sh   # regenerate frontend/openapi.json + typed API client from the backend's live OpenAPI schema
uv run prek install -f            # install git hooks (prek = Rust pre-commit alternative)
uv run prek run --all-files       # run all hooks manually (typos, ruff, mypy, SDK generation, zizmor)
```

The `generate-frontend-sdk` pre-commit hook regenerates the client automatically whenever `backend/**` changes — after editing backend routes/schemas, either let the hook run on commit or run `scripts/generate-client.sh` manually before relying on frontend types.

Dependency changes (`pyproject.toml`, `uv.lock`, `package.json`, `pnpm-lock.yaml`) from external contributors require a Discussion/Issue first (see `CONTRIBUTING.md`).

## Backend architecture

Each backend feature is a **module** under `backend/src/modules/<name>/` (`auth`, `users`, `friends`, `servers`, `channels`, `chats`, `messages`). All 7 modules follow the same layered, CQRS/mediator layout:

```
modules/<name>/
  domain/
    entities/          rich domain objects (plain classes subclassing shared.domain.Entity/
                        AggregateRoot — not Pydantic), one per file (e.g. `user.py`, `server.py`)
    entities/dtos.py   plain `@dataclass(frozen=True, kw_only=True)` DTOs: persistence create/
                        update payloads, read-model/query-result DTOs, facade DTOs (e.g.
                        `UserDTO`, `ChannelDTO`). Never Pydantic — see "Pydantic is transport-only"
                        below.
    repository Protocols, unit-of-work ABC, exceptions (LumiereError subclasses), enums
  application/
    commands/          one <Name>Command (frozen kw-only dataclass, data-only) + a separate
                        <Name>CommandHandler class with `async def handle(command) -> Result[T, E]`
    queries/            same shape: <Name>Query dataclass + <Name>QueryHandler
  infrastructure/
    persistence/        SQLAlchemy ORM models, repository implementations, model<->entity mappers
    <name>_unit_of_work_impl.py   concrete UnitOfWork wiring repositories to one AsyncSession
  public/
    facade.py            the module's boundary for OTHER modules: a `<Name>Facade` Protocol
                        (structurally typed, so consumers can fake it in unit tests without
                        building a real mediator/DB) plus a concrete impl and a
                        `build_<name>_facade(session[, stack])` factory. Present on modules that
                        at least one other module depends on (`users`, `channels`, `chats`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twpro8/lumiere](https://github.com/twpro8/lumiere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

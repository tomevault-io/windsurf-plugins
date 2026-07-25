---
trigger: always_on
description: provides `get_by_id`, `list_all`, `save`, `update`, `delete`, `save_batch`;
---

# Geti Backend Agent Guide

Component guide for `application/backend/` — the FastAPI `geti` service. Read this
together with the repo-wide `../../AGENTS.md` and the matching skill
`.agents/skills/geti-backend-dev/`.

## What This Component Is

- FastAPI application server named `geti`.
- Python 3.13, FastAPI, SQLAlchemy 2 (async where applicable), Pydantic v2,
  Alembic, `loguru`.
- Consumes the local `getitune` library (`../../library`) for training,
  inference, export, and quantization via the `getitune[cpu|xpu|cuda]` extras.
- Packaged with PyInstaller (`geti.spec`).

## App Layout (`app/`)

| Module / package      | Responsibility |
| --------------------- | -------------- |
| `main.py`             | FastAPI app entry point: registers routers, middleware, exception handlers, static files. |
| `lifecycle.py`        | Lifespan context: startup (DB init, services, workers, scheduler) and shutdown. |
| `settings.py`         | `pydantic-settings` config loaded from environment/`.env` (paths, ports, DB URL, limits). |
| `cli.py`              | Click CLI for DB ops (`init_db`, `migrate`, `check_db`, …) and helpers. |
| `create_openapi.py`   | Generates the OpenAPI spec JSON consumed by the UI. |
| `scheduler.py`        | Orchestrates worker processes and multiprocessing queues. |
| `api/`                | HTTP layer: `routers/`, `schemas/` (Pydantic DTOs), `dependencies.py`, `validators.py`. |
| `services/`           | Business logic; orchestrates repositories and workflows. |
| `repositories/`       | Data access; generic `BaseRepository` + per-resource repos wrapping ORM models. |
| `models/`             | Domain/business models (Pydantic), separate from ORM models. |
| `db/`                 | SQLAlchemy ORM (`schema.py`), engine/session (`engine.py`), migration mgmt (`migration.py`). |
| `alembic/`            | Alembic env and `versions/` migration scripts. |
| `core/`               | Core infrastructure: `jobs/` (queue/controller/exec), `run/`, `logging/`, core models. |
| `execution/`          | Long-running job implementations: `training/`, `dataset_export/`, `dataset_import/`, `quantization/` (lazy-imported builders). |
| `workers/`            | Multiprocessing workers for real-time tasks (inference, stream loading, dispatch). |
| `stream/`, `webrtc/`  | Real-time video streaming and WebRTC connection handling. |
| `datumaro_converter/` | Dataset format conversion (Geti internal ↔ COCO/VOC/… via Datumaro). |
| `supported_models/`   | Recommended model architectures/manifests by task type. |
| `utils/`              | Shared utilities. |

## Layered Architecture (import-linter)

Enforced by import-linter contracts in `pyproject.toml`. Inbound request flow:

```
app.api  →  app.services  →  app.repositories  →  app.db
```

- `app.api` may import `app.services`; not the reverse.
- `app.services` may import `app.repositories`; not the reverse.
- `app.repositories` may import `app.db`; not the reverse.
- Within the API layer: `app.api.routers` → `app.api.schemas` → `app.api.dependencies`.
- Model layering: `app.api.schemas` → `app.models` → `app.core.models`.

Keep these three model layers distinct:

- **API schemas** (`app/api/schemas/`): Pydantic request/response DTOs.
- **Domain models** (`app/models/`): pure business objects used by services.
- **ORM models** (`app/db/schema.py`): SQLAlchemy tables.

Never return raw ORM models from a router — convert to a Pydantic schema.

## API Layer

- Routers live in `app/api/routers/`, one file per resource (e.g. `projects.py`,
  `datasets.py`, `models.py`, `pipelines.py`, `media.py`, `jobs.py`, `sources.py`,
  `sinks.py`, `system.py`, `webrtc.py`, `training_configurations.py`, …).
- Routers are registered in `main.py` via `app.include_router(...)`.
- Dependencies (`app/api/dependencies.py`) provide `Depends(...)` injectors for the
  DB session, services, scheduler, and data/job directories. App-wide singletons
  live on `request.app.state` (set during startup).
- Keep handlers thin — delegate to services.

## Database Layer

- SQLite database at `application/backend/data/geti.db`.
- ORM models in `app/db/schema.py`: a `DeclarativeBase` with `created_at`/`updated_at`,
  a `BaseID` abstract base with a UUID text primary key, then per-resource tables.
- Engine/session in `app/db/engine.py` (uses `NullPool` for multiprocessing
  friendliness; foreign-key pragma enabled; context manager commits on success,
  rolls back on error).
- **Schema changes require an Alembic migration** under `app/alembic/versions/`.
  Manage with the CLI (`init_db`, `migrate`, `check_db`).

## Services & Repositories

- **Repositories** (`app/repositories/`): a generic `BaseRepository[ModelType]`
  provides `get_by_id`, `list_all`, `save`, `update`, `delete`, `save_batch`;
  per-resource repos subclass it (e.g. `ProjectRepository(BaseRepository[ProjectDB])`).
  Repos are CRUD only — no business rules.
- **Services** (`app/services/`): orchestrate repositories, enforce business rules,
  coordinate related creates/deletes, and return domain models. Custom exceptions
  (`ResourceNotFoundError`, `ResourceInUseError`, `ResourceWith*AlreadyExistsError`)
  express business failures.

Example flow for projects: `routers/projects.py` (`ProjectView`/`ProjectCreate`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-edge-platform/geti](https://github.com/open-edge-platform/geti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

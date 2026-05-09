---
trigger: always_on
description: This project uses **pyright** (not mypy) for type checking. Always run `pyright` to verify type correctness after changes to Python files. Never suggest or run mypy.
---

# Claude Code Project Guidelines

## Type Checking

This project uses **pyright** (not mypy) for type checking. Always run `pyright` to verify type correctness after changes to Python files. Never suggest or run mypy.

```bash
# Backend type checking
cd backend && uv run pyright <file>

# Frontend type checking
cd frontend && npm run type-check
```

## Linting

- **Backend**: Uses `ruff` for linting and formatting
- **Frontend**: Uses `eslint` for linting, `prettier` for formatting

```bash
# Backend linting
cd backend && uv run ruff check <file>

# Frontend linting
cd frontend && npm run lint
```

## Background Worker (SAQ)

The backend includes a background job queue powered by [SAQ](https://github.com/tobymao/saq) with PostgreSQL as the queue backend (no Redis required).

### Running the Worker

```bash
# Development
cd backend && uv run saq src.worker.worker_settings

# Production (Docker) — already configured in docker-compose.yml as `worker` service
saq src.worker.worker_settings
```

### How It Works

- **Job Queue**: SAQ manages job enqueueing, dequeuing (via Postgres LISTEN/NOTIFY), retries, and cancellation. SAQ creates its own tables (`saq_jobs`, `saq_stats`, `saq_versions`) automatically.
- **JobBatch**: A domain entity (`src/domain/jobs/entities/job_batch.py`) tracks groups of related SAQ jobs. For example, generating prereading for all chapters of a book creates one `JobBatch` with N individual SAQ jobs.
- **Worker Process**: Runs as a separate process (`src/worker.py`) using the same Docker image with a different entrypoint command. Creates a fresh DB session per task to avoid sharing sessions across concurrent coroutines.
- **Task Handlers**: Task logic lives in `src/infrastructure/jobs/tasks/`. Each handler class receives dependencies via constructor injection. The worker builds these handlers with fresh sessions per invocation.
- **Batch Progress**: Uses atomic SQL increments (not read-modify-write) to safely update batch progress from concurrent tasks.
- **Concurrency**: Controlled via `WORKER_CONCURRENCY` env var (default: 5).

### Adding New Job Types

1. Add a new value to `JobBatchType` enum in `src/domain/jobs/entities/job_batch.py`
2. Create a task handler in `src/infrastructure/jobs/tasks/`
3. Create a top-level async task function in `src/worker.py` and register it in `worker_settings["functions"]`
4. Create an enqueue use case in `src/application/jobs/use_cases/`
5. Wire it through the DI container (`src/containers/jobs.py`)

### API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/jobs/books/{book_id}/prereading` | Enqueue batch prereading generation |
| `GET` | `/api/v1/jobs/batches/{batch_id}` | Get batch status |
| `DELETE` | `/api/v1/jobs/batches/{batch_id}` | Cancel a batch |

## Architecture: Domain-Driven Design with Hexagonal/Dependency Injection Patterns

This backend follows DDD (Domain-Driven Design) with hexagonal architecture and dependency injection. **These boundaries are critical and must be strictly enforced:**

### Layer Responsibilities

1. **Domain Layer** (`backend/src/domain/`)
   - Contains domain entities, value objects, and domain services
   - **MUST NOT** depend on ORM models, Pydantic schemas, or any infrastructure
   - Pure business logic only
   - Example: `Book`, `Chapter`, `XPoint` (value objects)

2. **Application Layer** (`backend/src/application/`)
   - Contains use cases (application services)
   - Works with **domain entities**, NOT ORM models
   - Uses repository protocols (interfaces) via dependency injection
   - **MUST NOT** import from infrastructure layer
   - **MUST NOT** return Pydantic schemas - return domain entities

3. **Infrastructure Layer** (`backend/src/infrastructure/`)
   - Contains repository implementations
   - ORM models live **ONLY** here
   - Responsible for all ORM-to-domain and domain-to-ORM conversion
   - Implements repository protocols defined in domain layer

4. **Routers/API Layer** (`backend/src/routers/`)
   - HTTP endpoints
   - Uses use cases via dependency injection
   - Converts between domain entities and Pydantic schemas
   - **MUST NOT** use ORM models directly
   - **MUST NOT** contain business logic

### Critical Architectural Rules

1. **ORM Models**
   - ORM models belong **ONLY** in the infrastructure/repository layer
   - **NEVER** import or use ORM models in:
     - Routers
     - Application services (use cases)
     - Domain layer
   - Repositories are responsible for converting between ORM and domain entities

2. **Domain Entities vs ORM Models**
   - Domain entities are in `backend/src/domain/`
   - ORM models are in `backend/src/infrastructure/`
   - Domain entities must NOT depend on SQLAlchemy or ORM models
   - Application services work with domain entities, not ORM models

3. **Value Objects and Pydantic Schemas**
   - Domain value objects (e.g., `XPoint`, `ISBN`) must be converted to primitives before passing to Pydantic models
   - Do NOT pass raw value objects to Pydantic schema fields
   - Example:
     ```python
     # ✗ WRONG
     schema = BookSchema(xpoint=book.xpoint)  # XPoint is a value object

     # ✓ CORRECT

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crossbill-Highlights/crossbill-web](https://github.com/Crossbill-Highlights/crossbill-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

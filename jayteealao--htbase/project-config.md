---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HTBase is a web archiving service that provides a REST API for saving web pages using multiple archival methods (monolith, SingleFile, screenshot, PDF, readability). It wraps archiving commands in `ht` for live terminal preview, stores metadata in PostgreSQL, and provides optional AI-powered summarization and entity extraction.

## Architecture

### Core Components

**FastAPI Server** (`app/server.py`)
- Main application entry point with lifespan context manager
- Registers archivers on `app.state.archivers` during startup
- Mounts API routes via `app/api/` and web UI routes via `app/web/`
- Serves archived files statically under `/files` mount

**Archivers** (`app/archivers/`)
- All archivers extend `BaseArchiver` and implement `archive(url, item_id) -> ArchiveResult`
- Registration order in server.py matters for "all" pipeline: readability runs first (DOM dump reused by monolith), then monolith/singlefile-cli, finally screenshot/pdf
- Each archiver saves to `{DATA_DIR}/{item_id}/{archiver_name}/output.{ext}`

**HTRunner** (`app/core/ht_runner.py`)
- Wraps shell commands in `ht` terminal for live preview at port 7681
- Uses stdin/stdout JSON communication protocol
- Thread-safe command serialization via `self.lock`
- All ht interactions should use `.send_command()` or `.send_command_and_snapshot()`

**Database** (`app/db/`)
- PostgreSQL backend (SQLAlchemy + psycopg driver)
- `archived_urls` table enforces URL uniqueness (single row per URL)
- `archive_artifact` tracks individual archiver runs (one row per archiver per URL)
- `url_metadata` stores readability-extracted metadata (title, byline, text, etc.)
- `article_summaries`, `article_entities`, `article_tags` for AI-powered semantic analysis
- Connection string built in `AppSettings.database_url` from DB_HOST/DB_PORT/DB_NAME/DB_USER/DB_PASSWORD env vars

**Task Managers** (`app/task_manager/`)
- `BackgroundTaskManager`: Generic queue-based background worker (ABC with `.process()` method)
- `ArchiverTaskManager`: Processes batch archiving tasks; supports requeue for failed/pending artifacts
- `SummarizationTaskManager`: Generates summaries/tags/entities using HuggingFace TGI backend
- Task managers are started during server lifespan and run in daemon threads

**Configuration** (`app/core/config.py`)
- Environment-based settings via pydantic-settings (loads from .env file)
- Key settings: DATA_DIR, DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD, ENABLE_SUMMARIZATION, START_HT, SKIP_EXISTING_SAVES
- Access via `get_settings()` (lru_cached singleton)

**Frontend** (`frontend/`)
- React + TypeScript + Vite + TailwindCSS
- React Router for navigation, TanStack Query for data fetching
- Communicates with backend via axios

### Data Flow

1. POST `/save/{archiver}` receives SaveRequest (url, id)
2. URL reachability checked; 404s recorded immediately as failed
3. For each archiver: check if already saved (if SKIP_EXISTING_SAVES=true), else insert pending artifact
4. Archiver runs via ht_runner.send_command_and_snapshot()
5. Result recorded in archive_artifact table with success/exit_code/saved_path
6. If archiver is in SUMMARY_SOURCE_ARCHIVERS and ENABLE_SUMMARIZATION=true, enqueue summarization task
7. SummarizationTaskManager generates summary/tags/entities and stores in semantic tables

## Development Commands

### Docker

Build and start services:
```bash
docker compose up --build
```

Watch mode (rebuilds on file changes):
```bash
docker compose watch
```

View logs:
```bash
docker compose logs -f
```

Stop services:
```bash
docker compose down
```

### Testing

Run unit tests:
```bash
pytest tests/unit
```

Run integration tests (requires PostgreSQL):
```bash
# Set DB connection env vars first
export DB_HOST=127.0.0.1 DB_PORT=5432 DB_NAME=htbase DB_USER=postgres DB_PASSWORD=postgres
alembic upgrade head  # Run migrations
pytest tests/integration
```

Run with coverage:
```bash
pytest --cov=app --cov-report=html tests/
```

Run single test file:
```bash
pytest tests/unit/test_models.py -v
```

### Database Migrations

Create new migration:
```bash
alembic revision --autogenerate -m "description"
```

Apply migrations:
```bash
alembic upgrade head
```

Rollback one migration:
```bash
alembic downgrade -1
```

View migration history:
```bash
alembic history
```

**Note**: Database URL is constructed automatically from environment variables in `alembic/env.py` using AppSettings.database_url property.

### Frontend

Install dependencies:
```bash
cd frontend && npm install
```

Development server (with hot reload):
```bash
cd frontend && npm run dev
```

Build for production:
```bash
cd frontend && npm run build
```

Lint:
```bash
cd frontend && npm run lint
```

Format:
```bash
cd frontend && npm run format
```

### Manual Testing

Save a page via monolith:
```bash
curl -X POST http://localhost:8000/save/monolith \
  -H 'Content-Type: application/json' \
  -d '{"id":"test123","url":"https://example.com"}'
```

Save via all archivers:
```bash
curl -X POST http://localhost:8000/save/all \
  -H 'Content-Type: application/json' \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jayteealao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->

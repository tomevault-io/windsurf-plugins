---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kanchi is a Celery task monitoring system with a Python FastAPI backend and Nuxt.js frontend. The backend monitors Celery events via message broker and provides real-time WebSocket updates to the frontend.

## Development Commands

### Quick Start (from root directory)
```bash
# Start both backend and frontend in development mode
make dev

# View unified logs (backend and frontend)
make logs

# Start backend only
make backend

# Start frontend only
make frontend
```

### Backend (from `agent/` directory)
```bash
# Run the backend server (recommended)
poetry run python app.py

# Alternative legacy entry point
poetry run python main.py

# Run with custom broker
poetry run python app.py --broker amqp://guest:guest@localhost:5672//

# Development with auto-reload
poetry run python main.py --reload

# Linting and formatting
poetry run black .
poetry run ruff check .
poetry run mypy .

# Database migrations (Alembic)
poetry run alembic current              # Check current migration version
poetry run alembic upgrade head         # Apply all pending migrations
poetry run alembic revision --autogenerate -m "Description"  # Create new migration
```

### Frontend (from `frontend/` directory)
```bash
# Development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Generate static site
npm run generate

# Regenerate types from backend OpenAPI
npx swagger-typescript-api generate -p http://localhost:8765/openapi.json -o app/src/types -n api.ts --modular
```

### Testing Environment (from `scripts/test-celery-app/`)
```bash
# Start test environment (RabbitMQ, Redis, Workers)
make start

# Generate test tasks
make test-simple    # Simple tasks
make test-mixed     # Mixed load for 60 seconds  
make test-stress    # 1000 tasks stress test
make test-failing   # Failing tasks

# Monitor services
make monitor        # Open RabbitMQ and Flower UIs
make logs           # Show all service logs
make status         # Check service health
```

### Docker Deployment
```bash
# Build and run with Docker
docker build -t kanchi .
docker run -p 8765:8765 -p 3000:3000 kanchi

# Or use Docker Compose (from root)
docker-compose up

# With custom broker URL
docker run -p 8765:8765 -p 3000:3000 -e CELERY_BROKER_URL=amqp://user:pass@host:5672// kanchi
```

## Architecture

### Backend Structure (`agent/`)
- **`app.py`**: Main FastAPI application with WebSocket support and integrated frontend serving
- **`main.py`**: Legacy entry point (redirects to FastAPI app)
- **`monitor.py`**: Core Celery event monitoring via `CeleryEventMonitor`
- **`database.py`**: SQLite database manager with SQLAlchemy async support
- **`models.py`**: Pydantic data models for tasks, workers, events
- **`database_models.py`**: SQLAlchemy database models
- **`api/`**: REST API routes organized by resource (tasks, workers, websockets)
- **`services/`**: Business logic services for task and worker management
- **`connection_manager.py`**: WebSocket connection management
- **`event_handler.py`**: Celery event processing and broadcasting
- **`config.py`**: Configuration management

### Frontend Structure (`frontend/`)
- **Framework**: Nuxt 4 (Vue 3) with TypeScript
- **UI**: TailwindCSS + Radix UI components (reka-ui)
- **State**: Pinia stores for centralized state management
- **Types**: Auto-generated from backend OpenAPI schema in `app/src/types/`
- **Services**: Centralized API service layer using generated types

### Key Patterns
- **Type Safety**: All API interactions use auto-generated TypeScript types
- **Real-time Updates**: WebSocket broadcasts for live task monitoring
- **Persistence**: SQLite database for task history and worker state
- **Background Jobs**: Async tasks for orphan detection and cleanup
- **Integrated Serving**: FastAPI serves both API and built frontend

## Database

Uses SQLAlchemy with Alembic migrations for schema management. Supports:
- **SQLite** (default, for development)
- **PostgreSQL** (recommended for production)

### Features
- Task persistence and history
- Worker status tracking
- Orphan task detection and retry batches
- Comprehensive indexing for performance

### Migration System
- **Tool**: Alembic for database migrations
- **Auto-run**: Migrations execute automatically on app startup
- **BYOD**: Bring-Your-Own-Database via `DATABASE_URL` environment variable

Database models defined in `agent/database.py`.

## Logging

Kanchi uses a unified logging system that captures logs from both backend and frontend in a single file. **This feature is only available in development mode.**

### Enabling Logging
Set the `DEVELOPMENT_MODE` environment variable to enable unified logging:
```bash
export DEVELOPMENT_MODE=true
```

The `make dev` command automatically enables development mode.

### Log File
- **Location**: `agent/kanchi.log`
- **Format**: Timestamped logs with source prefix `[BACKEND]` or `[FRONTEND]`
- **Behavior**: Cleaned on backend startup (only in development mode)

### Viewing Logs
```bash
# Tail logs (last 100 lines and follow)
make logs

# Or manually
tail -n 100 -f agent/kanchi.log
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getkanchi/kanchi](https://github.com/getkanchi/kanchi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Project: Aegra — Open Source LangGraph Backend (Agent Protocol Server)

## Development Commands

### Environment Setup

```bash
# Install dependencies
uv sync

# Activate virtual environment (IMPORTANT for migrations)
source .venv/bin/activate

# Start database
docker compose up postgres -d

# Apply migrations
python3 scripts/migrate.py upgrade
```

### Running the Application

**Option 1: Docker (Recommended for beginners)**

```bash
# Start everything (database + migrations + server)
docker compose up aegra
```

**Option 2: Local Development (Recommended for advanced users)**

```bash
# Start development server with auto-reload
uv run uvicorn src.agent_server.main:app --reload

# Start with specific host/port
uv run uvicorn src.agent_server.main:app --host 0.0.0.0 --port 4242 --reload

# Start development database
docker compose up postgres -d
```

### Testing

```bash
# Run all tests
uv run pytest

# Run specific test file
uv run pytest tests/test_api/test_assistants.py

# Run tests with async support
uv run pytest -v --asyncio-mode=auto

# Run with coverage
uv run pytest --cov=src --cov-report=html

# Health check endpoint test
curl http://localhost:4242/health
```

### Database Management

```bash
# Database migrations (using our custom script)
python3 scripts/migrate.py upgrade
python3 scripts/migrate.py revision -m "description"
python3 scripts/migrate.py revision --autogenerate -m "description"

# Check migration status
python3 scripts/migrate.py current
python3 scripts/migrate.py history

# Reset database (development)
python3 scripts/migrate.py reset

# Start database
docker compose up postgres -d
```

### Code Quality (Optional - not currently configured)

```bash
# If ruff is added to dependencies, use:
# uv run ruff check .
# uv run ruff format .

# If mypy is added, use:
# uv run mypy src --cache-dir .mypy_cache
```

## High-Level Architecture

Aegra is an **Agent Protocol server** that acts as an HTTP wrapper around **official LangGraph packages**. The key architectural principle is that LangGraph handles ALL state persistence and graph execution, while the FastAPI layer only provides Agent Protocol compliance.

### Core Integration Pattern

**Database Architecture**: The system uses a hybrid approach:

- **LangGraph manages state**: Official `AsyncPostgresSaver` and `AsyncPostgresStore` handle conversation checkpoints, state history, and long-term memory
- **Minimal metadata tables**: Our SQLAlchemy models only track Agent Protocol metadata (assistants, runs, thread_metadata)
- **URL format difference**: LangGraph requires `postgresql://` while our SQLAlchemy uses `postgresql+asyncpg://`

### Configuration System

**aegra.json**: Central configuration file that defines:

- Graph definitions: `"weather_agent": "./graphs/weather_agent.py:graph"`
- Authentication: `"auth": {"path": "./auth.py:auth"}`
- Dependencies and environment

**auth.py**: Uses LangGraph SDK Auth patterns:

- `@auth.authenticate` decorator for user authentication
- `@auth.on.{resource}.{action}` for resource-level authorization
- Returns `Auth.types.MinimalUserDict` with user identity and metadata

### Database Manager Pattern

**DatabaseManager** (src/agent_server/core/database.py):

- Initializes both SQLAlchemy engine and LangGraph components
- Handles URL conversion between asyncpg and psycopg formats
- Provides singleton access to checkpointer and store instances
- Auto-creates LangGraph tables via `.setup()` calls
- **Note**: Database schema is now managed by Alembic migrations (see `alembic/versions/`)

### Graph Loading Strategy

Agents are Python modules that export a compiled `graph` variable:

```python
# graphs/weather_agent.py
workflow = StateGraph(WeatherState)
# ... define nodes and edges
graph = workflow.compile()  # Must export as 'graph'
```

### FastAPI Integration

**Lifespan Management**: The app uses `@asynccontextmanager` to properly initialize/cleanup LangGraph components during FastAPI startup/shutdown.

**Health Checks**: Comprehensive health endpoint tests connectivity to:

- SQLAlchemy database engine
- LangGraph checkpointer
- LangGraph store

### Authentication Flow

1. HTTP request with Authorization header
2. LangGraph SDK Auth extracts and validates token
3. Returns user context with identity, permissions, org_id
4. Resource handlers filter data based on user context
5. Multi-tenant isolation via user metadata injection

## Key Dependencies

- **langgraph**: Core graph execution framework
- **langgraph-checkpoint-postgres**: Official PostgreSQL state persistence
- **langgraph-sdk**: Authentication and SDK components
- **psycopg[binary]**: Required by LangGraph packages (not asyncpg)
- **FastAPI + uvicorn**: HTTP API layer
- **SQLAlchemy**: For Agent Protocol metadata tables only
- **alembic**: Database migration management
- **asyncpg**: Async PostgreSQL driver for SQLAlchemy
- **greenlet**: Required for async SQLAlchemy operations

## Authentication System

The server uses environment-based authentication switching with proper LangGraph SDK integration:

**Authentication Types:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pessini/langgraph-skills-agent](https://github.com/pessini/langgraph-skills-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

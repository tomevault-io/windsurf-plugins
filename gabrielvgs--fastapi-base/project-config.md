---
trigger: always_on
description: FastAPI Base is a production-ready backend template built with FastAPI, SQLAlchemy, Alembic, Redis caching, Celery, and PostgreSQL. The project uses Docker Compose for local development and provides comprehensive tooling for code quality, testing, and database migrations.
---

# FastAPI Base Project

FastAPI Base is a production-ready backend template built with FastAPI, SQLAlchemy, Alembic, Redis caching, Celery, and PostgreSQL. The project uses Docker Compose for local development and provides comprehensive tooling for code quality, testing, and database migrations.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Installation

- Install `uv` package manager: `pip install uv`
- Install `pre-commit`: `pip install pre-commit`
- Ensure Docker and Docker Compose are available
- Copy `.env.example` to `.env` and configure environment variables

### Bootstrap and Setup

1. **Install dependencies** (takes 10-30 seconds):

   ```bash
   cd fastapi-base/
   uv sync
   ```

2. **Set up pre-commit hooks**:

   ```bash
   uvx pre-commit install
   ```

3. **Configure environment**:

   ```bash
   cp .env.example .env
   # Edit .env with appropriate database and Redis URLs
   ```

### Docker-Based Development

**IMPORTANT**: Docker builds may fail in sandboxed environments due to TLS certificate issues. If you encounter `invalid peer certificate: UnknownIssuer` errors, this is a known limitation and the instructions will note alternative approaches.

1. **Build and run with Docker Compose** (takes 2-5 minutes if successful):

   ```bash
   make build    # NEVER CANCEL: May take up to 5 minutes in normal environments
   ```

2. **Start services**:

   ```bash
   make up       # Starts all services (FastAPI, PostgreSQL, Redis, Celery, Beat)
   ```

3. **Access the application**:
   - API: `http://localhost:8666/v1/ping`
   - Documentation: `http://localhost:8666/docs`
   - OpenAPI: `http://localhost:8666/v1/`

### Native Development (Alternative to Docker)

If Docker builds fail due to network restrictions:

1. **Install dependencies**:

   ```bash
   cd fastapi-base/
   uv sync
   ```

2. **Run linting and formatting** (takes 5-10 seconds each):

   ```bash
   uv run ruff check src/        # Linting - very fast
   uv run black --check .        # Code formatting check
   uv run isort --check-only .   # Import sorting check
   uv run mypy src/              # Type checking - takes 5-10 seconds
   ```

3. **Fix linting/formatting issues**:

   ```bash
   uv run black .                # Apply black formatting
   uv run isort .                # Fix import sorting
   ```

4. **Run tests** (takes 5-10 seconds for unit tests):

   ```bash
   # Note: Full tests require running PostgreSQL and Redis
   POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test" REDIS_URL="redis://redis:6379" uv run pytest tests/ -v
   ```

## Development Workflow

### Making Code Changes

1. **Always start with code quality checks**:

   ```bash
   cd fastapi-base/
   uv run ruff check src/
   uv run mypy src/
   ```

2. **Make your changes** in the appropriate directory:
   - API routes: `src/api/v1/`
   - Business logic: `src/core/`
   - Database models: `src/models/`
   - Repository patterns: `src/repositories/`
   - Pydantic schemas: `src/schemas/`

3. **Apply formatting** after changes:

   ```bash
   uv run black .
   uv run isort .
   ```

4. **Validate changes**:

   ```bash
   uv run ruff check src/
   uv run mypy src/
   POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test" REDIS_URL="redis://redis:6379" uv run pytest tests/test_routes.py::test_health -v
   ```

### Adding New API Endpoints

1. **Create new route** in `src/api/v1/`
2. **Add corresponding tests** in `tests/`
3. **Update schemas** in `src/schemas/` if needed
4. **Add database models** in `src/models/` if needed
5. **Follow the health endpoint pattern** for caching and error handling

### Database Schema Changes

1. **Modify models** in `src/models/`
2. **Generate migration**:

   ```bash
   # With Docker (if available):
   make alembic-make-migrations "description of change"

   # Alternative: modify Alembic directly
   # Note: Requires running PostgreSQL instance
   ```

### Understanding the Architecture

- **FastAPI Application**: Entry point in `src/main.py`
- **API Versioning**: Routes organized under `src/api/v1/`
- **Dependency Injection**: Common dependencies in `src/api/deps.py`
- **Configuration Management**: `src/core/config.py` with Pydantic settings
- **Database Layer**: SQLAlchemy models and repository pattern
- **Caching**: Redis-based caching with FastAPI-Cache
- **Background Tasks**: Celery worker configuration
- **Testing**: Pytest with async support and fixtures

### Available Make Commands

**Docker-based commands** (may fail in sandboxed environments):

- `make build`: Build and start all services
- `make up`: Start existing services
- `make down`: Stop all services
- `make bash`: Connect to the FastAPI container

**Database commands** (require running PostgreSQL):

- `make alembic-init`: Initialize first migration
- `make alembic-migrate`: Apply migrations
- `make alembic-make-migrations`: Create new migration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GabrielVGS/fastapi-base](https://github.com/GabrielVGS/fastapi-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

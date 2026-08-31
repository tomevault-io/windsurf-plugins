---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Development
```bash
make dev                 # Start both backend (port 8000) and frontend (port 5173)
make dev-backend         # Start FastAPI backend only
make dev-frontend        # Start React frontend only
make install             # Install all dependencies (backend + frontend)
make setup               # Initial setup: install + run migrations
```

### Testing & Code Quality
```bash
make test                # Run all tests
make test-backend        # Run backend pytest tests
make test-frontend       # Run frontend vitest tests
make lint                # Run all linters
make format              # Format all code
make check               # Run lint + tests (CI check)
```

### Database
```bash
make db-upgrade          # Apply database migrations
make db-migrate MESSAGE="description"  # Create new migration
make db-history          # View migration history
```

### Single Test Execution
```bash
# Backend
cd backend && uv run pytest tests/test_specific_file.py::test_function -v

# Frontend
cd frontend && npm test -- specific_test_file
```

## High-Level Architecture

This project uses a **3-tier architecture with Adapter pattern** for database abstraction:

```
API Layer (FastAPI routes)
    ↓
Service Layer (Facade pattern)
    ↓
Adapter Registry (Factory pattern)
    ↓
Database Adapters (PostgreSQL, MySQL, etc.)
```

### Key Architectural Patterns

1. **Adapter Pattern**: Each database type has a dedicated adapter implementing `DatabaseAdapter` ABC
   - `app/adapters/base.py` - Abstract base class with 7 required methods
   - `app/adapters/postgresql.py` - PostgreSQL implementation
   - `app/adapters/mysql.py` - MySQL implementation
   - Adding new database: Create adapter file, register in registry, update enum

2. **Facade Pattern**: `DatabaseService` provides high-level interface coordinating adapters, validators, and business logic
   - Located: `app/services/database_service.py`
   - Global instance: `database_service` used throughout API layer

3. **Factory Pattern**: `DatabaseAdapterRegistry` manages adapter lifecycle and connection pools
   - Located: `app/adapters/registry.py`
   - Global instance: `adapter_registry` initialized with all supported databases

### Data Flow

**Query Execution:**
1. API receives request → `app/api/v1/queries.py`
2. Calls `execute_query_with_service()` → wraps `database_service.execute_query()`
3. Service validates SQL → `sql_validator.py`
4. Service gets adapter from registry → `adapters/registry.py`
5. Adapter executes query using database-specific driver
6. Result returned through service layer → API response

**Metadata Extraction:**
1. API request → `app/api/v1/databases.py`
2. Calls `fetch_metadata()` → `app/services/metadata.py`
3. Checks cache (24hr TTL) in SQLite database
4. If stale/missing: calls `database_service.extract_metadata()`
5. Adapter queries information_schema
6. Results cached and returned

## Code Conventions

### Backend (Python/FastAPI)
- **Type hints**: Required for all function signatures
- **Async/await**: All database operations are async
- **Error handling**: Use `SqlValidationError` for SQL issues, `HTTPException` for API errors
- **Logging**: Use `logging` module, not `print()`
- **API routes**: Use `sqlmodel` for request/response models in `app/models/schemas.py`
- **Database models**: Use SQLModel classes in `app/models/database.py`, `app/models/query.py`

### Frontend (React/TypeScript)
- **Components**: Functional components with hooks
- **State**: React hooks (useState, useEffect)
- **API calls**: Axios via `app/services/`
- **Types**: TypeScript interfaces in `app/types/`
- **Styling**: Tailwind CSS classes (camelCase for dynamic styles)
- **Design System**: MotherDuck-inspired (sunbeam yellow #FFDE00, 2px black borders)

### Naming Conventions
- **Python**: `snake_case` for functions/variables, `PascalCase` for classes
- **TypeScript/React**: `PascalCase` for components, `camelCase` for functions/variables
- **API endpoints**: `kebab-case` for URL parameters (e.g., `/api/v1/dbs/{name}`)
- **Database**: `snake_case` for table/column names

## Important File Locations

### Backend Structure
```
backend/
├── app/
│   ├── api/v1/              # API route definitions
│   │   ├── databases.py     # Database connection endpoints
│   │   ├── queries.py       # Query execution endpoints
│   │   └── export.py        # Data export endpoint (CSV/JSON)
│   ├── adapters/            # Database abstraction layer
│   │   ├── base.py         # DatabaseAdapter ABC + data types
│   │   ├── registry.py     # Adapter factory and pool management
│   │   ├── postgresql.py   # PostgreSQL adapter
│   │   └── mysql.py        # MySQL adapter
│   ├── services/           # Business logic
│   │   ├── database_service.py  # Main facade
│   │   ├── sql_validator.py    # SQL validation/transformation
│   │   ├── export_service.py   # Query result export (CSV/JSON)
│   │   ├── nl2sql.py           # Natural language to SQL
│   │   └── metadata.py         # Metadata caching
│   ├── models/             # Data models

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yangyoung-max/Ai-Demo-Tool](https://github.com/Yangyoung-max/Ai-Demo-Tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FastAPI application implementing **Clean Architecture** (also known as Hexagonal Architecture or Onion Architecture) with strict dependency rules, Repository pattern, and Unit of Work pattern. The architecture enforces that dependencies point INWARD - outer layers depend on inner layers, never the reverse.

## Architecture Layers

The codebase is organized into four concentric layers, from innermost to outermost:

### 1. Domain Layer (`app/domain/`)
- **Purpose**: Core business logic, completely framework-agnostic
- **Contains**: Entities, repository interfaces, domain services interfaces, domain exceptions
- **Dependencies**: NONE - this layer has zero external dependencies
- **Key principle**: Pure Python classes with business rules and validations

**Example**: `User` entity (`app/domain/entities/user.py`) validates email format and name requirements without knowing about databases or HTTP.

### 2. Application Layer (`app/application/`)
- **Purpose**: Use case orchestration and business workflows
- **Contains**: Application services, DTOs, application exceptions
- **Dependencies**: Only depends on Domain layer abstractions
- **Key principle**: Services depend on interfaces (e.g., `IUnitOfWork`, `IPasswordHasher`), not implementations

**Example**: `UserService` (`app/application/services/user_service.py`) orchestrates user creation by calling repositories and domain services through their interfaces.

### 3. Infrastructure Layer (`app/infrastructure/`)
- **Purpose**: Concrete implementations of domain interfaces
- **Contains**: Database models, repository implementations, security implementations, configuration
- **Dependencies**: Depends on Domain layer (implements its interfaces)
- **Key principle**: Implements abstractions defined in the domain layer

**Example**: `UserRepository` (`app/infrastructure/repositories/user_repository_impl.py`) implements `IUserRepository` using SQLAlchemy.

### 4. Presentation Layer (`app/presentation/`)
- **Purpose**: HTTP API and external interface
- **Contains**: FastAPI routers, dependencies (composition root), exception handlers
- **Dependencies**: Depends on Application and Infrastructure layers
- **Key principle**: Translates HTTP requests to application service calls

**Example**: User router (`app/presentation/api/v1/users.py`) receives HTTP requests and delegates to `UserService`.

## Key Patterns

### Unit of Work Pattern
- **Interface**: `IUnitOfWork` (`app/domain/repositories/unit_of_work.py`)
- **Implementation**: `UnitOfWork` (`app/infrastructure/repositories/unit_of_work_impl.py`)
- **Purpose**: Manages database transactions and provides access to all repositories within a transactional boundary
- **Usage**: Services receive a `uow_factory` callable that creates UoW instances, ensuring each operation gets its own transaction

```python
async with self._uow_factory() as uow:
    user = await uow.users.get_by_id(user_id)
    # ... business logic ...
    await uow.commit()
```

### Repository Pattern
- **Base Interface**: `IRepository[T]` (`app/domain/repositories/base.py`)
- **Entity-specific Interface**: `IUserRepository` extends `IRepository[User]`
- **Purpose**: Abstracts data persistence, allowing domain and application layers to be database-agnostic

### Dependency Injection
- **Composition Root**: `app/presentation/dependencies.py` is where ALL concrete implementations are wired up
- **Pattern**: Inner layers define interfaces, outer layers provide implementations
- **Testing**: Use fakes from `tests/fakes/` directory (e.g., `FakePasswordHasher`, `FakeUnitOfWork`)

### Domain Entity Validation
Entities validate their own invariants in `__post_init__` and domain methods:
- **Structural violations**: Raise `InvalidEntityStateException` (entity cannot exist)
- **Business rule violations**: Raise `BusinessRuleViolationException` (operation not allowed)

## Database & Migrations

**Database**: PostgreSQL with async support (asyncpg)

### Alembic Commands
```bash
# Create a new migration
alembic revision --autogenerate -m "description"

# Apply migrations
alembic upgrade head

# Revert last migration
alembic downgrade -1

# View migration history
alembic history
```

**Note**: Alembic is configured to use sync driver. Update `alembic.ini` line 87 with your database URL if needed.

## Running the Application

### Development Server
```bash
# Run with auto-reload
fastapi dev app/main.py

# Or using uvicorn directly
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Environment Setup
Copy `.env.example` to `.env` and configure:
- `SECRET_KEY`: Must be at least 32 characters (required)
- `DB_*`: Database connection parameters
- `ENVIRONMENT`: Set to `dev`, `test`, or `prod`

All settings are defined in `app/infrastructure/config/settings.py`.

## Testing

### Run Tests
```bash
# All tests
pytest

# Unit tests only (fast, no database)
pytest -m unit

# Integration tests (use database)
pytest -m integration

# With coverage
pytest --cov=app --cov-report=html

# Run specific test file
pytest tests/unit/test_user_service.py -v
```

### Test Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vanthao03596/clean-architecture-fastapi](https://github.com/vanthao03596/clean-architecture-fastapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

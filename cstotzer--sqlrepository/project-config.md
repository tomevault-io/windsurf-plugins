---
trigger: always_on
description: A Python repository pattern implementation for SQLAlchemy and SQLModel, inspired by Spring Data JPA repositories. Provides type-safe, zero-boilerplate CRUD operations with sync and async support.
---

# CLAUDE.md — sqlrepository

A Python repository pattern implementation for SQLAlchemy and SQLModel, inspired by Spring Data JPA repositories. Provides type-safe, zero-boilerplate CRUD operations with sync and async support.

**Current version**: derived from git tags (see `git describe --tags`)
**License**: MIT
**Python**: >= 3.11

---

## Development Commands

This project uses **uv** for dependency management.

```bash
# Install all dependencies (including optional groups)
uv sync --all-groups

# Run all tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=sqlrepository --cov-report=term-missing

# Run a specific test file or test
uv run pytest tests/sqlalchemy/test_repository.py -v
uv run pytest tests/sqlalchemy/test_repository.py::test_find_by_id -v

# Lint
uv run ruff check src tests

# Format
uv run ruff format src tests

# Type check
uv run pyright src
```

---

## Project Structure

```
sqlrepository/
├── .github/
│   ├── instructions.md              # Detailed contributor guide
│   ├── CICD.md                      # CI/CD pipeline documentation
│   ├── copilot-skills/
│   │   ├── chore.instructions.md
│   │   └── release.instructions.md
│   ├── actions/
│   │   └── setup-env/action.yml     # Composite action: setup-uv + uv sync
│   ├── dependabot.yml               # Automated github-actions version updates
│   └── workflows/
│       ├── ci.yml                   # Quality checks + tests on push/PR
│       ├── release.yml              # Release on vX.Y.Z tag push
│       └── update-deps.yml          # Weekly uv lock --upgrade PR
├── src/sqlrepository/
│   ├── __init__.py                  # Exports: Repository, AsyncRepository, EntityType, IdType
│   ├── core.py                      # Sync repository (SQLAlchemy DeclarativeBase)
│   ├── asyncio.py                   # Async repository (SQLAlchemy AsyncSession)
│   └── sqlmodel.py                  # SQLModel-specific Repository and AsyncRepository
├── tests/
│   ├── conftest.py                  # Shared fixtures and JSON data loading
│   ├── resources/
│   │   ├── artists.json             # Test artist data (6 records)
│   │   └── albums.json              # Test album data
│   ├── sqlalchemy/
│   │   ├── conftest.py              # SQLAlchemy fixtures (in-memory SQLite)
│   │   ├── models.py                # Test models: Artist, Album
│   │   ├── repositories.py          # Test repository implementations
│   │   ├── test_repository.py       # Sync CRUD tests
│   │   ├── test_async_repository.py # Async CRUD tests
│   │   ├── test_special_cases.py    # Edge cases / error handling
│   │   └── test_generics.py         # Generic type parameter tests
│   └── sqlmodel/
│       ├── conftest.py              # SQLModel fixtures
│       ├── models.py                # SQLModel test models
│       ├── repositories.py          # SQLModel test repositories
│       ├── test_repository.py       # Sync SQLModel tests
│       ├── test_async_repository.py # Async SQLModel tests
│       ├── test_special_cases.py
│       └── test_generics.py
├── pyproject.toml                   # Build config, dependencies, tool config
├── uv.lock                          # Locked dependencies
├── README.md                        # User-facing documentation with examples
├── trivy.yaml                       # Security scanning config
└── LICENSE
```

---

## Architecture

### Module Layout

| File | Purpose |
|------|---------|
| `core.py` | `BaseRepository` mixin + `Repository[EntityType, IdType]` for SQLAlchemy `DeclarativeBase` models |
| `asyncio.py` | `BaseAsyncRepository` mixin + `AsyncRepository[EntityType, IdType]` for `AsyncSession` |
| `sqlmodel.py` | `Repository[EntityType, IdType]` and `AsyncRepository[EntityType, IdType]` for SQLModel models |
| `__init__.py` | Re-exports `Repository` (from `core`), `AsyncRepository` (from `asyncio`), `EntityType`, `IdType` |

> **Note**: `sqlmodel.py` exports its own `Repository` and `AsyncRepository` — these are different classes from the ones in `core.py`/`asyncio.py`. SQLModel users import from `sqlrepository.sqlmodel`, not from `sqlrepository` directly.

### Type Inference via `__init_subclass__`

Both sync and async repositories automatically extract the model type from the generic parameters:

```python
class ArtistRepository(Repository[Artist, int]):
    pass  # cls._model_type is automatically set to Artist
```

`__init_subclass__` inspects `__orig_bases__` to extract the first generic argument.

### Available CRUD Methods

All repository classes expose the same interface:

| Method | Description |
|--------|-------------|
| `save(entity)` | Insert or merge entity; flushes to get generated IDs |
| `save_all(entities)` | Save a collection of entities |
| `find_by_id(id)` | Return entity or `None` |
| `find_all()` | Return all entities |
| `find_all_by_id(ids)` | Return entities matching any of the given IDs |
| `exists_by_id(id)` | Return `bool` (issues `SELECT 1 LIMIT 1`; never loads the entity) |
| `count()` | Return total row count |
| `delete(entity)` | Remove a specific entity |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cstotzer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

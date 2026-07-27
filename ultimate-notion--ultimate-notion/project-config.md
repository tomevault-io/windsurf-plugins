---
trigger: always_on
description: Ultimate-Notion is a pythonic, high-level API for Notion. It provides a comprehensive interface to interact
---

# Project Configuration

## 1. Project Overview

Ultimate-Notion is a pythonic, high-level API for Notion. It provides a comprehensive interface to interact
with Notion's API, allowing users to manage pages, databases (data sources), blocks, and more.

**Current Major Work**: Migration to Notion API 2025-09-03

- Migrating from "database" terminology to "data source" for what Notion previously called databases
- Introducing a NEW "Database" concept that represents collections of data sources
- Adopting `ds` abbreviation for data sources, reserving `db` for the new database collections

## 2. Key Technologies & Conventions

### Core Technologies

- **Python**: 3.10+ with full type hints
- **Pydantic**: For data validation and object modeling
- **notion-client**: Official Notion SDK (currently 3.1 for API 2026-03-11)
- **VCR.py**: For recording and replaying HTTP interactions in tests

### Code Conventions

- **Type Safety**: Full static type checking required (mypy and ty, via `hatch run lint:typing`)
- **Code Style**: Ruff for linting and formatting
- **Naming Conventions**:
  - `ds` = data source (what was previously called "database")
  - `db` = database collection (NEW concept in API 2025-09-03)
  - Use snake_case for functions, variables, and methods
  - Use PascalCase for classes
  - Use UPPER_CASE for constants

### Git Workflow

- **NEVER use `git add -A` or `git add .`** - always specify exact file paths
- Always use conventional commit messages
- Include co-authorship footer for AI-assisted commits
- Use feature branches for major work

## 3. Architectural Principles

### Two-Layer Design

**Low-Level Layer (`obj_api/`):**

- Direct mapping to Notion API objects
- Pydantic models for all API types
- Minimal business logic
- Used by high-level layer

**High-Level Layer:**

- Pythonic, user-friendly API
- Wraps low-level objects with `DataObject` pattern
- Provides schema definitions, queries, views
- Session-based interaction model

### Key Patterns

**DataObject Wrapper Pattern:**

```python
class DataSource(DataObject[obj_blocks.DataSource], wraps=obj_blocks.DataSource):
    """High-level wrapper for data source objects"""
```

**Schema Binding:**

- Schemas bind to data sources using `bind_ds()`
- Properties are defined using `PropType` descriptors
- Automatic property mapping and validation

**Type Guards:**

- Use `TypeIs` for proper type narrowing
- Prefer `isinstance()` checks for simple cases
- Type guards: `is_ds()`, `is_page()`, etc.

## 4. Navigation Architecture

### Source Code Structure

```shell
src/ultimate_notion/
├── obj_api/              # Low-level API layer
│   ├── blocks.py         # Block type definitions (DataSource, Database, Page, etc.)
│   ├── endpoints.py      # API endpoint wrappers (DataSourcesEndpoint, etc.)
│   ├── objects.py        # Reference objects (DataSourceRef, PageRef, etc.)
│   ├── schema.py         # Property type definitions
│   └── query.py          # Query builders
├── session.py            # Session management (create_ds, search_ds, etc.)
├── database.py           # DataSource and Database classes
├── page.py               # Page class and operations
├── blocks.py             # High-level block wrappers
├── schema.py             # Schema definition and binding
├── query.py              # High-level query interface
├── view.py               # View/result set handling
├── props.py              # Property value types
└── adapters/             # Third-party integrations
    └── google/tasks/     # Google Tasks sync
```

### Key Files

- `session.py` (528 lines): Main entry point for API operations
- `database.py` (308 lines): DataSource and Database classes
- `schema.py` (1376 lines): Schema definition and property system
- `page.py` (664 lines): Page operations and management
- `blocks.py` (1840 lines): Block type wrappers
- `query.py` (656 lines): Query building and filtering

## 5. Testing

### Test Structure

```shell
tests/
├── cassettes/           # VCR cassettes for recorded API calls
├── conftest.py          # Test fixtures and configuration
├── test_*.py            # Test modules
└── obj_api/             # Low-level API tests
```

### Key Testing Commands

```bash
# Run all tests (using VCR cassettes)
hatch run test

# Run tests without VCR (live API calls)
hatch run vcr-off

# Rewrite VCR cassettes (use with CAUTION)
hatch run vcr-rewrite

# Drop all cassettes
hatch run vcr-drop-cassettes

# Run only VCR cassettes (no live API)
hatch run vcr-only
```

### Testing Best Practices

1. **VCR Usage**: Most tests use VCR to replay recorded API interactions
2. **Fixtures**: Use fixtures from `conftest.py` for common objects
3. **Never use dynamic values**: `datetime.now()` will be recorded and replayed
4. **Fixture scope**: Be careful with module/session scoped fixtures

### Common Test Fixtures

- `notion`: Fresh session without state (function scope)
- `notion_cached`: Cached session (module scope)
- `root_page`: Test parent page
- `contacts_db`, `task_db`, `wiki_db`: Pre-configured data sources
- `person`: Test user object

## 6. Important Files & Directories

### Configuration Files

- `pyproject.toml`: Project dependencies, scripts, and tool configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ultimate-notion/ultimate-notion](https://github.com/ultimate-notion/ultimate-notion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

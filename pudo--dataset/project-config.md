---
trigger: always_on
description: **dataset** is a lightweight Python library that makes reading and writing data in databases as simple as working with JSON files. It provides a simple abstraction layer on top of SQLAlchemy, removing most direct SQL statements without requiring a full ORM model.
---

# dataset: databases for lazy people

## Project Overview

**dataset** is a lightweight Python library that makes reading and writing data in databases as simple as working with JSON files. It provides a simple abstraction layer on top of SQLAlchemy, removing most direct SQL statements without requiring a full ORM model.

**Key Philosophy:** Simplicity over complexity. The library is designed for small to medium-scale datasets where you want database benefits without the overhead of complex ORM patterns.

## Repository Structure

```
dataset/
├── dataset/              # Main package
│   ├── __init__.py      # Main entry point with connect()
│   ├── database.py      # Database class - connection, transactions, queries
│   ├── table.py         # Table class - CRUD operations, schema management
│   ├── types.py         # SQLAlchemy type mapping and helpers
│   ├── util.py          # Utility functions (ResultIter, normalization, etc.)
│   └── chunked.py       # Chunked operations for large datasets
├── test/                # Test suite (pytest)
│   ├── conftest.py      # Test fixtures
│   ├── test_database.py
│   ├── test_table.py
│   └── test_row_type.py
├── docs/                # Sphinx documentation (RST format)
└── pyproject.toml       # Package configuration (PEP 621)
```

## Core Concepts

### 1. Database Connection
- Single entry point: `dataset.connect(url)` returns a `Database` instance
- Supports SQLite (default), PostgreSQL, and MySQL
- Connection pooling managed by SQLAlchemy
- Environment variable fallback: `DATABASE_URL`

### 2. Automatic Schema Management
- **Default behavior:** Tables and columns are created automatically on insert
- Can be disabled via `ensure_schema=False` parameter
- Column types are guessed from values (or explicitly specified)
- Thread-safe with locking for schema operations

### 3. CRUD Operations
- **insert()**: Add single row, returns primary key
- **insert_many()**: Bulk insert with chunking (default: 1000 rows)
- **update()**: Update rows matching keys
- **upsert()**: Insert or update based on key columns
- **delete()**: Delete rows matching filters
- **find()**: Query with filters, ordering, limit/offset
- **find_one()**: Return single row or None

### 4. Transaction Support
- Context manager: `with dataset.connect() as tx:`
- Manual control: `db.begin()`, `db.commit()`, `db.rollback()`
- Nested transactions supported
- SQLAlchemy 2.x "autobegin" semantics with explicit commit tracking

## Important Implementation Details

### Threading & Concurrency
- Each thread gets its own database connection (thread-local storage)
- Schema changes in transactions with multiple threads trigger warnings
- Locking strategy:
  - `self.lock` (RLock) protects metadata operations
  - Keep lock scope small for performance
  - Schema operations are fully synchronized

### Transaction Handling (SQLAlchemy 2.x Migration)
- The library supports both SQLAlchemy 1.4+ and 2.x
- Key change: SQLAlchemy 2.x uses "autobegin" (transactions start on first use)
- `db._auto_commit()` commits after writes when not in explicit transaction
- Transaction nesting tracked via `self.local.tx` stack

### Column & Table Name Normalization
- Case-insensitive column matching via `normalize_column_key()`
- Actual database names preserved via `_column_keys` mapping
- Tables and columns validated for safety

### Type System
- `db.types` provides shortcuts to SQLAlchemy types
- Type guessing from Python values: `types.guess(value)`
- Custom types via `types` parameter in insert/update
- MySQL-specific: text field indexing uses 10-char prefix

### Type Annotations
- The codebase passes `mypy --strict` and ships `py.typed` (PEP 561)
- Key type aliases in `util.py`:
  - `WriteRow = Mapping[str, SQLWriteValue]` — immutable input type for public API
  - `MutableRow = dict[str, SQLWriteValue]` — mutable type for internal use (queue items, return values from `_sync_columns`)
  - `OutRow = Mapping[str, Any]` — query result rows
  - `RowFactory = Callable[[Iterable[tuple[str, Any]]], OutRow]` — the `row_type` parameter type
- `ColumnType` in `types.py` is `TypeEngine[Any] | type[TypeEngine[Any]]` — used for `primary_type` and `create_column`
- Use `WriteRow` at public API boundaries, `MutableRow` internally where mutation happens
- `ensure_strings()` in `util.py` replaces `banal.ensure_list` — accepts `str | Iterable[str] | None`, returns `list[str]`
- `QueryError` (subclass of `DatasetError`) is raised for invalid filter values in `_generate_clause`

## Testing Strategy

### Test Infrastructure
- **Framework:** pytest
- **Fixtures:**
  - `db`: Function-scoped database connection with cleanup
  - `table`: Pre-populated weather table with test data
- **CI/CD:** GitHub Actions testing against SQLite, PostgreSQL, MySQL
- **Database Cleanup:** Explicit rollback + drop tables in teardown

### Running Tests
```bash
make test                    # Run all tests
pytest test/test_table.py    # Run specific test file
DATABASE_URL="postgresql://..." pytest  # Test against specific DB
```

### Test Database Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pudo/dataset](https://github.com/pudo/dataset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

---
trigger: always_on
description: Python library for distributed data loading of OpenHouse tables. Uses PyIceberg for Iceberg table access, DataFusion for query execution and SQL transforms, and SQLGlot for SQL transpilation across dialects.
---

# Claude Instructions for OpenHouse DataLoader

## Project Overview

Python library for distributed data loading of OpenHouse tables. Uses PyIceberg for Iceberg table access, DataFusion for query execution and SQL transforms, and SQLGlot for SQL transpilation across dialects.

## Common Commands

```bash
make sync           # Install dependencies (uses uv)
make check          # Run lint + format checks + typecheck (mypy)
make test           # Run unit tests (pytest)
make verify         # Run all checks and tests
make format         # Auto-format code (ruff)
make build          # Build package distributions
make package-check  # Validate built distributions with twine
make clean          # Clean build artifacts
```

## Workflows

When validating a change, always run both:

1. `make verify` — lint, format checks, typecheck, and unit tests
2. Integration tests against Docker OpenHouse — start the Docker services, then run `make integration-tests`. These test the dataloader end-to-end against a real OpenHouse instance and must pass before a change is considered correct.

Run `make format` before pushing to avoid CI formatting failures.

```bash
# From the repo root, start Docker services (once per session):
docker compose -f infra/recipes/docker-compose/oh-hadoop-spark/docker-compose.yml up -d

# From the dataloader directory:
make format
make verify
make integration-tests TOKEN_FILE=../../../tables-test-fixtures/tables-test-fixtures-iceberg-1.2/src/main/resources/dummy.token
```

## Integration Tests

Integration tests run inside a Docker container on the same network as the oh-hadoop-spark services. The `make integration-tests` target builds a test image and runs it automatically. Tables are created and populated via Spark SQL submitted through Livy.

1. Start the Docker services from the repo root:
   ```bash
   docker compose -f infra/recipes/docker-compose/oh-hadoop-spark/docker-compose.yml up -d
   ```
2. Wait for all services to be healthy (especially Livy and namenode), then run:
   ```bash
   make integration-tests TOKEN_FILE=../../../tables-test-fixtures/tables-test-fixtures-iceberg-1.2/src/main/resources/dummy.token
   ```

## Project Structure

```
src/openhouse/dataloader/
├── __init__.py              # Public API exports
├── data_loader.py           # Main API: OpenHouseDataLoader, DataLoaderContext
├── data_loader_split.py     # DataLoaderSplit (single callable split, yields RecordBatches)
├── catalog.py               # OpenHouseCatalog (REST-based PyIceberg Catalog impl)
├── filters.py               # Filter expression DSL (col(), always_true(), combinators)
├── datafusion_sql.py        # SQLGlot DataFusion dialect + SQL transpilation
├── table_identifier.py      # TableIdentifier dataclass
├── table_transformer.py     # TableTransformer ABC (SQL-based transforms with dialect)
├── udf_registry.py          # UDFRegistry ABC + NoOpRegistry default
├── _table_scan_context.py   # TableScanContext (shared scan state, pickle-safe)
└── _timer.py                # log_duration context manager
```

## Public API

Exported in `__init__.py`:
- `OpenHouseDataLoader` — Main entry point; accepts a `Catalog`, table coordinates, optional filters/columns/context
- `DataLoaderContext` — Execution context with optional `TableTransformer` and `UDFRegistry`
- `OpenHouseCatalog` — REST catalog that loads tables via the OpenHouse Tables Service API
- `OpenHouseCatalogError` — Error raised when catalog fails to load a table
- `col()` — Column reference for building filter expressions
- `always_true()` — Filter that matches all rows
- `to_sql()` — Render a filter as a SQL boolean expression (WHERE-clause predicate) for a target
- `SqlTarget` — Enum of supported SQL flavors (`SPARK`, `TRINO`, `DATA_FUSION`); used by `to_sql()` and `TableTransformer`

### Filter DSL (`filters.py`)

Build row filters using `col()` with comparison operators (`==`, `!=`, `>`, `>=`, `<`, `<=`) and predicates (`is_null()`, `is_not_null()`, `is_nan()`, `is_not_nan()`, `is_in()`, `is_not_in()`, `starts_with()`, `not_starts_with()`, `between()`). Combine with `&` (AND), `|` (OR), `~` (NOT). Filters are converted to PyIceberg expressions internally for partition pruning and file-level filtering.

`to_sql(filter, target=SqlTarget.SPARK)` renders a filter as a SQL boolean expression for the given target. Internally, `_filter_to_expr()` builds a single dialect-agnostic sqlglot AST that is rendered per target with `.sql(dialect=target.value)` — the same path the loader uses for its internal DataFusion query via `to_sql(filters, SqlTarget.DATA_FUSION)`. The backing sqlglot dialect string is an internal detail — callers select a `SqlTarget`, never a dialect string.

### Internal modules (not in `__init__.py`)

- `TableTransformer` — ABC for SQL-based table transforms; subclass must provide a `dialect` (a `SqlTarget`, e.g. `SqlTarget.SPARK`) and implement `transform()` returning SQL or `None`
- `UDFRegistry` / `NoOpRegistry` — ABC for registering DataFusion UDFs; `NoOpRegistry` is the default no-op

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linkedin/openhouse](https://github.com/linkedin/openhouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

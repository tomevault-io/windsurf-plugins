---
trigger: always_on
description: Enables adapters to work with Apache Iceberg and other external table formats.
---

# dbt Adapters — Development Guide

## Monorepo Structure

```
dbt-adapters/
├── dbt-adapters/          # Base framework and protocols
├── dbt-tests-adapter/     # Reusable test suite
├── dbt-postgres/          # PostgreSQL adapter (base for redshift)
├── dbt-redshift/          # Amazon Redshift adapter (extends postgres)
├── dbt-snowflake/         # Snowflake adapter
├── dbt-bigquery/          # Google BigQuery adapter
├── dbt-spark/             # Apache Spark / Databricks adapter
├── dbt-athena/            # AWS Athena adapter
└── .pre-commit-config.yaml
```

Dependency chain:
```
dbt-adapters (base)
├── dbt-postgres
│   └── dbt-redshift
├── dbt-snowflake
├── dbt-bigquery
├── dbt-spark
└── dbt-athena

dbt-tests-adapter → used by all adapters for testing
```

## Development Workflow

All commands run from the specific adapter directory (e.g. `cd dbt-redshift`).

```shell
# Prerequisites
pip install hatch changie pre-commit

# Initial setup (installs adapter + deps in editable mode)
hatch run setup

# Code quality (Black, Flake8, MyPy)
hatch run code-quality

# Unit tests (no database required)
hatch run unit-tests
hatch run unit-tests -- tests/unit/test_file.py::TestClass::test_method -v

# Integration tests (requires test.env)
hatch run integration-tests

# Changelog entry
changie new   # Categories: Breaking Changes, Features, Fixes, Under the Hood, Dependencies, Security

# IDE integration: use local .hatch virtualenv
hatch config set dirs.env.virtual .hatch
```

Never commit `test.env` credentials.

## Base Framework (`dbt-adapters`)

**Location:** `dbt-adapters/src/dbt/adapters/`

### Core Classes

| File | Class | Purpose |
|------|-------|---------|
| `base/impl.py` | `BaseAdapter` | Abstract base for all adapters |
| `base/connections.py` | `BaseConnectionManager` | Thread-safe connection pooling |
| `base/relation.py` | `BaseRelation` | Database relation representation |
| `base/column.py` | `Column` | Column metadata and type handling |
| `base/plugin.py` | `AdapterPlugin` | Plugin registration |
| `base/meta.py` | `@available` | Decorator for macro-accessible methods |
| `sql/impl.py` | `SQLAdapter` | SQL-specific base (extends BaseAdapter) |
| `sql/connections.py` | `SQLConnectionManager` | SQL connection patterns |

### Key Supporting Systems

- **`contracts/`** — `Credentials`, `Connection`, `RelationConfig` dataclasses
- **`capability.py`** — `Capability` enum and `CapabilityDict` for feature declaration
- **`cache.py`** — `RelationsCache` for performance optimization
- **`catalogs/`** — External catalog integrations (Iceberg, etc.)
- **`record/`** — Execution recording for replay/testing

### Global Macros (`include/global_project/macros/`)

- `materializations/` — table, view, incremental, snapshot, seed
- `relations/` — DDL for tables, views, columns
- `utils/` — Common utility macros

## Testing (`dbt-tests-adapter`)

**Location:** `dbt-tests-adapter/src/dbt/tests/adapter/`

### Pattern

```python
from dbt.tests.adapter.basic import BaseSimpleMaterializations

class TestSimpleMaterializations(BaseSimpleMaterializations):
    pass  # inherits all base tests

class TestCustomFeature(BaseIncremental):
    @pytest.fixture(scope="class")
    def models(self):
        return {"model.sql": "select 1 as id"}

    def test_custom_logic(self, project):
        results = run_dbt(["run"])
        assert len(results) == 1
```

### Required Tests (`basic/`)

`BaseSimpleMaterializations`, `BaseEmpty`, `BaseEphemeral`, `BaseSnapshotTimestamp`, `BaseSnapshotCheck`, `BaseIncremental`, `BaseGenericTests`, `BaseSingularTests`, `BaseAdapterMethod`, `BaseValidateConnection`, `BaseDocsGenerate`

### Optional Tests

`incremental/`, `constraints/`, `grants/`, `materialized_view/`, `relations/`, `python_model/`, `unit_testing/`, `concurrency/`

## Adapter Structure

All adapters follow this layout:

```
dbt-{adapter}/
├── src/dbt/
│   ├── adapters/{adapter}/
│   │   ├── __init__.py           # Plugin registration
│   │   ├── connections.py        # ConnectionManager & Credentials
│   │   ├── impl.py               # Adapter implementation
│   │   ├── relation.py           # Relation class
│   │   └── column.py             # Column type handling (if needed)
│   └── include/{adapter}/macros/ # SQL macro overrides
├── tests/unit/
├── tests/functional/
├── pyproject.toml
├── hatch.toml
└── test.env.example
```

Macros override defaults by prefixing with adapter name:
```sql
{% macro redshift__list_relations_without_caching(schema_relation) %}
    -- Redshift-specific implementation
{% endmacro %}
```

---

## dbt-postgres

Foundation SQL adapter; other adapters extend it.

**Key files:** `impl.py`, `connections.py` (psycopg2), `relation.py`, `column.py`

**Capabilities:** All constraint types (CHECK, NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY), indexes, materialized views.

```sql
-- Indexes
{{ config(indexes=[
  {'columns': ['col_a'], 'type': 'btree'},
  {'columns': ['col_a', 'col_b'], 'unique': true},
]) }}

-- Materialized views
{{ config(materialized='materialized_view') }}

-- Unlogged tables
{{ config(materialized='table', unlogged=true) }}
```

**Credentials:** `host`, `port` (5432), `user`, `password`, `database`, `schema`, `sslmode`, `sslcert`, `sslkey`, `sslrootcert`, `search_path`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbt-labs/dbt-adapters](https://github.com/dbt-labs/dbt-adapters) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pg_lake integrates Apache Iceberg and data lake files (Parquet, CSV, JSON) into PostgreSQL, enabling PostgreSQL to function as a lakehouse system. The architecture consists of two main components:
- **PostgreSQL with pg_lake extensions**: Handles query planning, transaction boundaries, and orchestration
- **pgduck_server**: A separate multi-threaded process that implements the PostgreSQL wire protocol and delegates computation to DuckDB's columnar execution engine

Users connect only to PostgreSQL. The pg_lake extensions transparently delegate data scanning and computation to pgduck_server (running DuckDB) when appropriate, while maintaining full transactional guarantees.

## Build Commands

### First-time build
```bash
# Install vcpkg dependencies (required once)
export VCPKG_VERSION=2025.10.17
git clone --recurse-submodules --branch $VCPKG_VERSION https://github.com/Microsoft/vcpkg.git
./vcpkg/bootstrap-vcpkg.sh
./vcpkg/vcpkg install azure-identity-cpp azure-storage-blobs-cpp azure-storage-files-datalake-cpp openssl
export VCPKG_TOOLCHAIN_PATH="$(pwd)/vcpkg/scripts/buildsystems/vcpkg.cmake"

# Build and install all extensions and pgduck_server
make install
```

### Subsequent builds
```bash
# Fast build that skips rebuilding DuckDB if already built
make install-fast
```

### Component-specific builds
```bash
# Build/install individual extensions
make install-pg_lake_iceberg
make install-pg_lake_table
make install-pgduck_server

# Build all extensions (top-level meta extension)
make install-pg_lake
```

## Running pg_lake

### Required setup
```sql
-- In postgresql.conf:
shared_preload_libraries = 'pg_extension_base'

-- Connect to PostgreSQL and create extensions:
CREATE EXTENSION pg_lake CASCADE;
-- This installs: pg_extension_base, pg_map, pg_extension_updater,
-- pg_lake_engine, pg_lake_iceberg, pg_lake_table, pg_lake_copy, pg_lake

-- Set default location for Iceberg tables:
SET pg_lake_iceberg.default_location_prefix TO 's3://your-bucket/pglake';
```

### Starting pgduck_server
```bash
# Start pgduck_server (must be running for pg_lake to work)
pgduck_server

# With options:
pgduck_server --memory_limit '8GB' --cache_dir /tmp/cache --init_file_path /path/to/init.sql

# pgduck_server listens on port 5332 (unix socket /tmp by default)
# You can connect directly to pgduck_server for debugging:
psql -p 5332 -h /tmp
```

## Testing

The project uses **pytest** for all regression testing (not traditional PostgreSQL SQL regression tests).

### Running all tests
```bash
# Install test dependencies (first time only)
pipenv install --dev

# Run all local tests
make check

# Run end-to-end tests (requires S3/cloud access)
make check-e2e

# Run upgrade tests
make check-upgrade

# Run all tests (local + e2e)
make check  # includes check-local and check-e2e
```

### Running tests for specific components
```bash
# Test specific extension
make check-pg_lake_table
make check-pg_lake_iceberg
make check-pgduck_server

# Run isolation tests
make check-isolation_pg_lake_table
```

### Running installcheck
```bash
# Start pgduck_server with test configuration
pgduck_server --init_file_path pgduck_server/tests/test_secrets.sql --cache_dir /tmp/cache &

# Run installcheck (tests against installed extensions)
make installcheck

# Run installcheck for specific component
make installcheck-pg_lake_table
```

### Running individual pytest tests
```bash
cd pg_lake_table
PYTHONPATH=../test_common pipenv run pytest -v tests/pytests/test_specific.py
PYTHONPATH=../test_common pipenv run pytest -v tests/pytests/test_specific.py::test_function_name
```

### Testing with PostgreSQL regression suite
```bash
# Run PostgreSQL's own tests with pg_lake extensions loaded
export PG_REGRESS_DIR=/path/to/postgres/src/test/regress

# Run tests
make installcheck-postgres PG_REGRESS_DIR=$PG_REGRESS_DIR
make installcheck-postgres-with_extensions_created PG_REGRESS_DIR=$PG_REGRESS_DIR
```

## Extension Architecture

pg_lake follows a **modular design** with interoperating components. Each extension has a specific responsibility:

### Extension dependency chain
```
pg_lake (meta-extension)
├── pg_lake_table (FDW for querying data lake files)
│   └── pg_lake_iceberg (Iceberg specification implementation)
│       └── pg_lake_engine (common module for pg_lake extensions)
│           ├── pg_extension_base (foundation for all extensions)
│           ├── pg_map (generic map type)
│           └── pg_extension_updater (automatic extension updates)
└── pg_lake_copy (COPY to/from data lake)
    └── pg_lake_engine

pg_lake_spatial (optional, depends on PostGIS)
pg_lake_benchmark (optional, for benchmarking)
```

### Core extensions
- **pg_extension_base**: Foundation for all extensions, provides common utilities
- **pg_extension_updater**: Automatically updates extensions on startup
- **pg_map**: Generic map/key-value type generator for semi-structured data
- **pg_lake_engine**: Common module shared by data lake extensions (depends on Apache Avro)
- **pg_lake_iceberg**: Full Iceberg v2 protocol implementation with transactional support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Snowflake-Labs/pg_lake](https://github.com/Snowflake-Labs/pg_lake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

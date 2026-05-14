---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DataFusion-DuckLake is a DataFusion extension that provides read-only access to DuckLake catalogs. DuckLake is an integrated data lake and catalog format that stores:
- **Metadata**: In SQL databases (DuckDB, SQLite, PostgreSQL, MySQL) as structured catalog tables
- **Data**: As Apache Parquet files on disk or object storage (S3, MinIO)

The extension integrates DuckLake with Apache DataFusion by implementing DataFusion's catalog and table provider interfaces.

## Commands

### Build and Test
```bash
# Build the project
cargo build

# Run all tests
cargo test

# Run a specific test
cargo test test_name

# Build and run the basic query example
cargo run --example basic_query -- <catalog.db> <sql>
```

## Architecture

### Core Components

The codebase follows a layered architecture with clear separation of concerns:

1. **MetadataProvider Layer** (`src/metadata_provider.rs`, `src/metadata_provider_duckdb.rs`)
   - Abstraction for querying DuckLake catalog metadata
   - `MetadataProvider` trait defines interface for listing schemas, tables, columns, and data files
   - Also provides individual lookup methods: `get_schema_by_name()`, `get_table_by_name()`, and `table_exists()`
   - `DuckdbMetadataProvider` implements the trait using DuckDB as the catalog backend
   - Executes SQL queries against standard DuckLake catalog tables (`ducklake_snapshot`, `ducklake_schema`, `ducklake_table`, `ducklake_column`, `ducklake_data_file`, `ducklake_delete_file`, `ducklake_metadata`)
   - Thread-safe: Uses a single shared connection protected by Mutex for efficiency
   - Supports delete files: `get_table_files_for_select()` returns data files with associated delete files

2. **DataFusion Integration Layer** (`src/catalog.rs`, `src/schema.rs`, `src/table.rs`)
   - Bridges DuckLake concepts to DataFusion's catalog system
   - `DuckLakeCatalog`: Implements `CatalogProvider`, uses dynamic metadata lookup (queries on every call to `schema()` and `schema_names()`)
   - `DuckLakeSchema`: Implements `SchemaProvider`, uses dynamic metadata lookup (queries on every call to `table()` and `table_names()`)
   - `DuckLakeTable`: Implements `TableProvider`, caches table structure and file lists at creation time
   - **No HashMaps**: Catalog and schema providers query metadata on-demand rather than caching

3. **Path Resolution** (`src/path_resolver.rs`)
   - Centralized utilities for parsing object store URLs and resolving hierarchical paths
   - `parse_object_store_url()`: Parses S3, file://, or local paths into ObjectStoreUrl and path components
   - `resolve_path()`: Resolves relative or absolute paths in the catalog hierarchy
   - `PathResolver`: Maintains base URL and path for hierarchical resolution (catalog -> schema -> table -> file)
   - Handles S3, MinIO, and local filesystem paths uniformly

4. **Delete File Filtering** (`src/delete_filter.rs`)
   - `DeleteFilterExec`: Custom execution plan that wraps Parquet scans and filters deleted rows
   - Implements MOR (Merge-On-Read) pattern for row-level deletes
   - Delete files contain `(file_path: VARCHAR, pos: INT64)` schema
   - Efficiently filters rows by position during query execution
   - Supports COUNT(*) optimization (zero-column batches)

5. **Type Mapping** (`src/types.rs`)
   - Converts DuckLake type strings to Arrow DataTypes
   - Handles basic types (integers, floats, strings, dates, timestamps)
   - Supports decimals with precision/scale parsing
   - Complex types (lists, structs, maps) return proper errors instead of silently failing
   - `build_arrow_schema()` constructs Arrow schemas from DuckLake column metadata

### Dynamic Metadata Lookup

The catalog uses a **pure dynamic lookup** approach with no caching at the catalog/schema level:

- **DuckLakeCatalog** (`catalog.rs`):
  - `schema_names()`: Queries `list_schemas()` on every call
  - `schema()`: Queries `get_schema_by_name()` on every call
  - `new()`: O(1) - only fetches snapshot ID and data_path

- **DuckLakeSchema** (`schema.rs`):
  - `table_names()`: Queries `list_tables()` on every call
  - `table()`: Queries `get_table_by_name()` on every call
  - `table_exist()`: Queries `table_exists()` on every call
  - `new()`: O(1) - just stores IDs and paths

- **DuckLakeTable** (`table.rs`):
  - Still caches table structure and file lists at creation time
  - This is necessary for query planning and execution

**Benefits**:
- O(1) memory usage regardless of catalog size
- Fast catalog startup (no upfront schema/table listing)
- Always fresh metadata (no stale cache issues)
- Simple implementation (no cache invalidation logic)

**Trade-offs**:
- Small query overhead per metadata lookup (acceptable for read-only DuckDB connections)
- Future optimization: Add optional caching layer via wrapper implementation

### Data Flow

When querying a DuckLake table:
1. User creates a `SessionContext` with a `RuntimeEnv` and registers a `DuckLakeCatalog`
2. User registers required object stores (S3, MinIO, etc.) with the `RuntimeEnv`
3. SQL query references table as `catalog.schema.table`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datafusion-contrib/datafusion-ducklake](https://github.com/datafusion-contrib/datafusion-ducklake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

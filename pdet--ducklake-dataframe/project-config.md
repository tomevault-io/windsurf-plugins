---
trigger: always_on
description: Handles quoted field names with escaped double-quotes.
---

# ducklake-dataframe

> **This file is the primary onboarding document for Claude instances working on this project.
> Keep it up to date as the codebase evolves. If you add a module, change an architectural
> decision, discover a new gotcha, or fix a tricky bug, update the relevant section here.**

## What this project is

A pure-Python Polars integration for [DuckLake](https://ducklake.select/) catalogs. It reads
DuckLake metadata from SQLite (via Python's stdlib `sqlite3`) or PostgreSQL (via `psycopg2`) and
scans the underlying Parquet data files through Polars' native Parquet reader. There is **no DuckDB
runtime dependency** -- DuckDB is only used in tests to create catalog fixtures.

Public API: `scan_ducklake()` (LazyFrame), `read_ducklake()` (DataFrame), and `DuckLakeCatalog`
(catalog inspection), exported from `ducklake_polars/__init__.py`.

## Architecture

```
src/ducklake_polars/
    __init__.py      Public API: scan_ducklake(), read_ducklake(), DuckLakeCatalog
    _backend.py      Backend adapters: SQLiteBackend, PostgreSQLBackend, create_backend()
    _catalog.py      Metadata reader (snapshots, tables, columns, files, stats, inlined data)
    _catalog_api.py  DuckLakeCatalog: high-level catalog inspection API
    _dataset.py      Polars PythonDatasetProvider implementation (DuckLakeDataset)
    _schema.py       DuckLake type string -> Polars DataType mapping
    _stats.py        Column statistics builder for file pruning
```

### Data flow

1. User calls `scan_ducklake(path, table)`.
2. `__init__.py` creates a `DuckLakeDataset` dataclass and passes it to Polars'
   private `PyLazyFrame.new_from_dataset_object()`.
3. Polars calls `DuckLakeDataset.schema()` to get the table schema, and
   `DuckLakeDataset.to_dataset_scan()` to get the actual data.
4. `to_dataset_scan()` opens a `DuckLakeCatalogReader` (read-only connection via the
   backend adapter), resolves the snapshot, finds data files/delete files, builds
   statistics, and returns a `scan_parquet(sources, ...)` LazyFrame.
5. Polars handles all query optimization (predicate pushdown, projection pushdown,
   file pruning via statistics, positional deletes via Iceberg-compatible delete files).

### Key Polars internals used

- `polars._plr.PyLazyFrame.new_from_dataset_object(dataset)` -- creates a LazyFrame
  from a Python object implementing the PythonDatasetProvider protocol.
- `scan_parquet(sources, missing_columns="insert", extra_columns="ignore",
  _table_statistics=..., _deletion_files=("iceberg-position-delete", ...))` --
  the underscore-prefixed params are private Polars APIs for statistics and deletes.
- These private APIs may change across Polars versions. If something breaks after
  a Polars upgrade, check these first.

### Column rename support

When `ALTER TABLE RENAME COLUMN` is used, old Parquet files still have the old physical column name.
DuckLake tracks renames via snapshot-versioned rows in `ducklake_column` (same `column_id`, different
`column_name` across snapshot boundaries). The rename logic:

1. `get_column_history()` retrieves all column definitions across all snapshots
2. `_has_renames()` checks if any column_id has multiple distinct names (fast path: no overhead if no renames)
3. If renames exist, files are grouped by their physical column names via `_group_files_by_rename_map()`
4. Each group is scanned separately with `scan_parquet`, collected eagerly, and old-name groups get `.rename()` applied
5. Groups are concatenated, written to a temp Parquet file, and returned as `scan_parquet(tmp_path)`

**Important**: The Polars dataset scan resolver only accepts bare Parquet SCAN nodes from
`to_dataset_scan()`. Returning `.rename()` (WITH_COLUMNS), `pl.concat()` (UNION), or
`df.lazy()` (DF) all fail with "unknown DSL when resolving python dataset scan". The temp
file workaround is the only viable approach. Temp files are cleaned up via `atexit` handlers.

### Partition pruning

DuckLake stores partition metadata in three tables: `ducklake_partition_info`, `ducklake_partition_column`,
`ducklake_file_partition_value`. For identity-transform partitions, partition values supplement the
`_table_statistics` DataFrame as a fallback when `ducklake_file_column_stats` is incomplete.
The logic is in `_build_partition_values_for_stats()` and integrated into `build_table_statistics()`.

### DuckLake metadata schema

The catalog database (SQLite or PostgreSQL) contains these tables (among others):

- `ducklake_metadata` -- key-value pairs, notably `data_path`
- `ducklake_snapshot` -- snapshot_id, schema_version, snapshot_time, next_file_id
- `ducklake_schema` -- schema_id, schema_name, path, begin_snapshot, end_snapshot
- `ducklake_table` -- table_id, table_name, schema_id, path, begin_snapshot, end_snapshot
- `ducklake_column` -- column_id, table_id, column_name, column_type, column_order,
  parent_column, begin_snapshot, end_snapshot, nulls_allowed
- `ducklake_data_file` -- data_file_id, table_id, path, record_count, begin_snapshot, end_snapshot
- `ducklake_delete_file` -- delete_file_id, data_file_id, table_id, path, begin_snapshot, end_snapshot
- `ducklake_file_column_stats` -- data_file_id, column_id, null_count, min_value, max_value
- `ducklake_partition_info` -- partition_id, table_id, begin_snapshot, end_snapshot

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pdet/ducklake-dataframe](https://github.com/pdet/ducklake-dataframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->

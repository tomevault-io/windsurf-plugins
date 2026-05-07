---
trigger: always_on
description: This is a DuckLake TPCH demo workshop that demonstrates lakehouse capabilities using DuckDB's DuckLake extension. The project uses **DuckDB CLI directly** (`duckdb -f file.sql`) instead of Python wrappers for maximum simplicity.
---

# DuckLake TPCH Workshop - Cursor Rules

## Project Overview
This is a DuckLake TPCH demo workshop that demonstrates lakehouse capabilities using DuckDB's DuckLake extension. The project uses **DuckDB CLI directly** (`duckdb -f file.sql`) instead of Python wrappers for maximum simplicity.

## Architecture

### Core Components
- **`Makefile`**: Commands that execute `duckdb -f scripts/*.sql` (Linux/macOS)
- **`scripts/*.sql`**: SQL scripts executed directly via `duckdb -f`
- **`generate_data.py`**: Standalone Python script for TPCH data generation
- **`load_small_files.py`**: Standalone Python script for loading files one at a time
- **`config/tpch.yaml`**: Configuration file for TPCH generation
- **`catalog/ducklake.ducklake`**: DuckLake catalog database (metadata storage)
- **`data/tpch/`**: Raw TPCH Parquet files (source data)
- **`data/lake/`**: DuckLake-managed partitioned Parquet files

### Key Technologies
- **DuckDB CLI**: Executes SQL files directly via `duckdb -f file.sql`
- **DuckLake Extension**: Provides lakehouse capabilities (partitioning, snapshots, time travel)
- **Python**: Only for data generation (`generate_data.py`) and file loading loops (`load_small_files.py`)
- **uv**: Python package manager (used for Python scripts)
- **Makefile**: Convenience wrapper for Unix systems (Windows users use DuckDB CLI directly)

## Core Principles (MANDATORY)

### 1. SQL Files Execute Directly with DuckDB CLI
**STRICT RULE**: All SQL runs directly via `duckdb -f scripts/file.sql`.
- ✅ ALWAYS: Use `duckdb -f scripts/file.sql` to execute SQL files
- ✅ SQL files are self-contained and executable
- ✅ SQL files use DuckDB variables (`SET VARIABLE` / `getvariable()`) for parameterization
- ❌ NEVER: Use Python to execute SQL files (except `load_small_files.py` which loops)

### 2. SQL Failures Abort Immediately
**STRICT RULE**: When SQL fails, DuckDB CLI aborts immediately.
- ✅ DuckDB CLI exits with non-zero exit code on SQL errors
- ✅ Makefile uses `set -e` to abort on errors
- ✅ No error suppression - SQL errors are fatal

### 3. Python Scripts Must Be Caveman Simple
**STRICT RULE**: Python scripts (`generate_data.py`, `load_small_files.py`) are minimal.
- ❌ NEVER: Use try/except blocks
- ❌ NEVER: Use complex if/else conditionals
- ✅ ALWAYS: Write linear, straightforward code
- ✅ Python is ONLY for: data generation and file loops - nothing else

### 4. Use DuckDB Variables, Not Python String Replacement
**STRICT RULE**: SQL files use DuckDB's `SET VARIABLE` and `getvariable()` for parameters.
- ✅ ALWAYS: Use `SET VARIABLE table_name = 'lineitem';` in SQL files
- ✅ ALWAYS: Use `getvariable('table_name')` to reference variables
- ✅ SQL files set defaults, users can override via `SET VARIABLE` before execution
- ❌ NEVER: Use Python string replacement for SQL parameters (except `load_small_files.py`)

## Critical Patterns & Gotchas

### 1. DuckDB Variable Usage
**CRITICAL**: Use DuckDB variables for parameterization.
```sql
-- Set variable with default
SET VARIABLE table_name = 'lineitem';

-- Use variable in query
SELECT * FROM lake.{getvariable('table_name')};

-- Or use in WHERE clause
WHERE table_name = getvariable('table_name');
```

**Variable Scopes**:
- `SET VARIABLE` (SESSION scope) - persists for the SQL file execution
- Variables can be set before executing SQL file: `duckdb -c "SET VARIABLE table_name = 'orders';" -f scripts/compaction.sql`

### 2. SQL File Execution Pattern
**Standard execution**:
```bash
duckdb -f scripts/bootstrap_catalog.sql
```

**With variable override**:
```bash
duckdb -c "SET VARIABLE table_name = 'orders';" -f scripts/compaction.sql
```

**Multiple statements**:
```bash
duckdb -c "SET VARIABLE older_than = INTERVAL '7 days';" -f scripts/expire_snapshots.sql
```

### 3. Fully Qualified Table Names
**IMPORTANT**: Use fully qualified table names (`lake.table_name`) in SQL files.
- Tables are defined in `bootstrap_catalog.sql` with schema `lake.`
- SQL files should reference tables as `lake.table_name`
- This is a SQL authoring best practice

### 4. File Loading Loop (load_small_files.py)
**ONLY EXCEPTION**: `load_small_files.py` uses Python to loop through files.
- This script loops through Parquet files and executes INSERT statements
- Each INSERT gets its own DuckDB connection
- This is the ONLY place where Python executes SQL statements

## File Structure Conventions

### SQL Files (`scripts/*.sql`)
- All SQL files go in `scripts/` directory
- Use fully qualified names: `lake.table_name`
- Include header comments explaining purpose
- Use `SET VARIABLE` for configurable parameters with defaults
- Structure: Comments → Variable Setup → Setup (INSTALL/LOAD/ATTACH/USE) → Main SQL → Cleanup

### Python Scripts
- **`generate_data.py`**: TPCH data generation (reads `config/tpch.yaml`)
- **`load_small_files.py`**: Loops through Parquet files and inserts them one at a time
- Both scripts are standalone and execute via `uv run python script.py`

### Makefile
- Provides convenience targets for Unix systems
- Each target executes `duckdb -f scripts/file.sql` or `uv run python script.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matsonj/ducklake-workshop](https://github.com/matsonj/ducklake-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->

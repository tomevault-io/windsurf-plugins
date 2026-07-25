---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Overview

DuckDB is a high-performance analytical database system designed to be fast, reliable, portable, and easy to use. It is an analytical database management system with a rich SQL dialect, vectorized execution engine, and columnar storage format.

## Build Commands

### Basic Build
```bash
make debug                   # Builds debug version with sanitizers and assertions
make reldebug                # Builds optimized release version with debug symbols
FORCE_DEBUG=1 make relassert # Builds optimized version with sanitizers and assertions
```

## Testing

### Running Tests
```bash
build/reldebug/test/unittest # Fast unit tests
```

### Running Specific Tests
```bash
# Run specific test file
build/reldebug/test/unittest test/sql/order/test_limit.test

# Run all tests including slow tests
build/reldebug/test/unittest "*"
```

It is recommended to use `make reldebug` and `build/reldebug/test/unittest` unless a good reason exists to use the debug build - the debug build is much slower than the reldebug build.

### Test File Format
Tests use the sqllogictest format (`.test` files). Example structure:
```sql
# name: test/sql/order/test_limit.test
# description: Test LIMIT keyword
# group: [order]

statement ok
CREATE TABLE test (a INTEGER, b INTEGER);

query I
SELECT a FROM test LIMIT 1
----
11

statement error
SELECT a FROM test LIMIT a
----
<REGEX>:Binder Error:.*not found.*
```

Test directives:
- `statement ok` - Statement should succeed
- `statement error` - Statement should fail
- `query I` - Query returning INTEGER column
- `query II` - Query returning two columns
- `----` - Separates query from expected results
- `<REGEX>:` - Expected error message pattern
- `require-env VAR` - Test requires environment variable

Slow tests should use `.test_slow` extension instead of `.test`.

## Code Formatting

```bash
make format-fix        # Format all code (clang-format + black)
make generate-files    # Generate files + format all code
```

Ensure you run formatting before committing.

## Extensive Testing / Making CI Work

Below is a set of tests that should be run in order to make sure a changeset passes extensive tests in CI. If the user is asking you to fix CI make sure that the below commands succeed.

```bash
make allunit
FORCE_DEBUG=1 FORCE_ASSERT=1 make reldebug && build/reldebug/test/unittest
make test_configs
make test_vector
```


## Architecture

### Query Execution Pipeline

```
SQL String
    ↓
[PARSER] - Uses a PEG parser to parse SQL into AST
    ↓
SQLStatement tree (ParsedExpression, TableRef objects)
    ↓
[PLANNER/BINDER] - Binds symbols to catalog, creates logical plan
    ↓
Logical Plan (LogicalOperator tree with bound Expressions)
    ↓
[OPTIMIZER] - Applies rule-based and cost-based optimizations
    ↓
Optimized Logical Plan
    ↓
[PHYSICAL PLAN GENERATOR] - Converts to physical operators
    ↓
Physical Plan (PhysicalOperator tree)
    ↓
[EXECUTOR] - Executes with vectorized, parallel pipelines
    ↓
Results
```

### Core Components

**Parser** (`src/parser/`)
- Converts SQL strings to Abstract Syntax Tree (AST)
- Uses a PEG-based parser
- The grammar is located in `*.gram` files and generated using `scripts/build_grammar.sh`
- Outputs: `SQLStatement`, `ParsedExpression`, `TableRef` objects
- Key subdirectories: `expression/`, `statement/`, `tableref/`, `peg/`

For more details on adding new grammar, see the README located at `src/parser/peg/README.md`. 
Each new grammar rule must have a corresponding transformer rule, located at `peg/transformer`.

**Planner** (`src/planner/`)
- Binds symbols to catalog entries and resolves types
- Creates logical query execution plan
- Key classes: `Binder`, `LogicalOperator`, bound `Expression` types
- Subdirectories: `binder/`, `expression/`, `subquery/`

**Optimizer** (`src/optimizer/`)
- Transforms logical plans without changing semantics
- Applies predicate pushdown, join ordering, expression rewriting, etc.
- Subdirectories: `join_order/`, `statistics/`, `rule/`, `pushdown/`

**Execution Engine** (`src/execution/`)
- Converts logical plan to physical plan and executes
- Push-based vectorized execution model
- Processes data in batches (typically 2048 rows)
- Key subdirectories: `operator/` (scan, join, filter, aggregate, etc.), `expression_executor/`

**Storage** (`src/storage/`)
- Manages persistent data storage and buffer management
- Block-based storage with compression
- Includes WAL (Write-Ahead Log) for durability
- Subdirectories: `buffer/`, `compression/`, `checkpoint/`, `table/`

**Catalog** (`src/catalog/`)
- Metadata management for tables, schemas, functions, types, etc.
- Single source of truth for database metadata
- Key classes: `Catalog`, `CatalogEntry`, `SchemaCatalogEntry`

**Transaction Manager** (`src/transaction/`)
- ACID transaction management with MVCC
- Coordinates concurrent access to data
- Key files: `transaction_manager.cpp`, `undo_buffer.cpp`, `wal_write_state.cpp`

**Parallel Execution** (`src/parallel/`)
- Multi-threaded execution with task scheduling
- Pipeline-based parallelism
- Key files: `executor.cpp`, `pipeline_executor.cpp`, `task_scheduler.cpp`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckdb/duckdb](https://github.com/duckdb/duckdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

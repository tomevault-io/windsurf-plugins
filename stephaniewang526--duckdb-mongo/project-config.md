---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# duckdb-mongo Agent Guidelines

This file provides guidance to AI coding agents working with code in this repository.

## What This Repo Is

A DuckDB extension that integrates MongoDB, enabling direct SQL queries over MongoDB collections without ETL. It uses the MongoDB C++ driver (via vcpkg) and extends DuckDB with a custom catalog, storage extension, and optimizer for query pushdown.

## Agent Skills

Reusable workflows in `.agents/skills/`. When a task matches one of these, read the corresponding file and follow its steps:

- **`.agents/skills/test.md`** — Use when asked to build, run tests, or verify changes. Covers MongoDB setup, incremental builds, and test execution.
- **`.agents/skills/repro-issue.md`** — Use when asked to investigate, reproduce, or fix a GitHub issue. Covers fetching the issue, root cause analysis, writing a failing test, and verifying the fix.
- **`.agents/skills/bump-duckdb-version.md`** — Use when asked to update the DuckDB submodule version. Covers updating submodules, fixing compilation, updating CI/CD and README, and running tests.

## Development Command Quick Reference

### Build Commands

- `make release` — Build the extension (release mode, output in `build/release/`)
- `make debug` — Build in debug mode
- `make reldebug` — Release build with debug symbols
- `cmake --build build/release --target mongo_loadable_extension` — Rebuild just the loadable extension
- `cmake --build build/release --target unittest` — Rebuild just the test binary

The build produces:
- `build/release/extension/mongo/mongo.duckdb_extension` — Loadable extension
- `build/release/test/unittest` — Test runner (embeds DuckDB + extension)

There is **no standalone DuckDB CLI binary** in this build. To interact with the extension manually, use a system-installed `duckdb` with `LOAD 'build/release/extension/mongo/mongo.duckdb_extension';`.

### Build Prerequisites

- CMake, C++17 compiler
- vcpkg with `mongo-cxx-driver` (run `bash scripts/setup-vcpkg.sh` if not set up)
- Set `VCPKG_TOOLCHAIN_PATH` if vcpkg isn't auto-detected (see `scripts/check-vcpkg.sh`)

### Testing Commands

**Tests require a running MongoDB instance with test data.** The full automated workflow:

```bash
bash test/run-tests-with-mongo.sh   # Starts Docker MongoDB, creates data, builds, runs tests
```

To run tests manually (when MongoDB is already running on localhost:27017):

```bash
# 1. Create test data (only needed once per MongoDB instance)
bash test/create-mongo-tables.sh
bash test/create-tpch-test-db.sh    # Optional: for TPC-H tests

# 2. Run all tests
MONGODB_TEST_DATABASE_AVAILABLE=1 make test_release

# 3. Run a single test file
MONGODB_TEST_DATABASE_AVAILABLE=1 build/release/test/unittest "test/sql/schema/schema.test"
```

**Without MongoDB**, all tests requiring `require-env MONGODB_TEST_DATABASE_AVAILABLE` will skip (which is most of them).

### Formatting

Code style is inherited from DuckDB (`.clang-format` symlinks to `duckdb/.clang-format`):
- LLVM-based style, 120-char line limit, tab indentation
- Run: `clang-format -i src/*.cpp src/**/*.cpp src/include/*.hpp`

## Code Architecture

### Source Organization

```
src/
├── include/                        # Header files
│   ├── mongo_table_function.hpp    # Core data structures (MongoScanData, MongoScanState)
│   ├── mongo_catalog.hpp           # MongoCatalog, MongoDefaultGenerator
│   ├── mongo_schema_entry.hpp      # Schema catalog entry
│   ├── mongo_instance.hpp          # MongoDB client singleton
│   ├── mongo_filter_pushdown.hpp   # Filter pushdown API
│   ├── mongo_expr_pushdown.hpp     # Expression pushdown API
│   ├── mongo_optimizer.hpp         # Custom optimizer
│   ├── mongo_secrets.hpp           # Secrets integration
│   └── ...
├── schema/
│   ├── mongo_schema_inference_helpers.cpp   # BSON→DuckDB type conversion, JSON normalization
│   └── mongo_schema_inference_internal.hpp  # Internal schema inference API
├── mongo_extension.cpp             # Extension entry point (registers everything)
├── mongo_table_function.cpp        # mongo_scan bind/init/execute
├── mongo_schema_inference.cpp      # Schema inference from documents
├── mongo_catalog.cpp               # Catalog (databases→schemas, collections→views)
├── mongo_optimizer.cpp             # Query pushdown optimizer
├── mongo_filter_pushdown.cpp       # WHERE clause → MongoDB query translation
├── mongo_expr_pushdown.cpp         # Complex expression pushdown
├── mongo_storage_extension.cpp     # ATTACH support
├── mongo_schema_entry.cpp          # Schema entry management
├── mongo_transaction.cpp           # Read-only transaction support
├── mongo_secrets.cpp               # CREATE SECRET support
└── mongo_clear_cache.cpp           # Cache invalidation
```

### Key Concepts

- **ATTACH flow**: `mongo_storage_extension.cpp` → creates `MongoCatalog` → `MongoDefaultGenerator` creates views backed by `mongo_scan` table function
- **Schema inference**: Samples documents (default 100), flattens nested docs with `_` separator (e.g., `address.city` → `address_city`), parent docs also exposed as VARCHAR/JSON columns
- **Query pushdown**: Custom optimizer (`mongo_optimizer.cpp`) rewrites DuckDB plans to push filters, projections, limits, and aggregations to MongoDB

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stephaniewang526/duckdb-mongo](https://github.com/stephaniewang526/duckdb-mongo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->

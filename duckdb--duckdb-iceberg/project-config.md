---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

## Scope and repository layout

This file applies to the repository root. `duckdb/` is an upstream submodule with its own `duckdb/AGENTS.md`; follow that file for work inside the submodule.

- `src/`: Iceberg extension C++ implementation and public headers.
- `test/sql/`: sqllogictests. REST-catalog coverage is under `test/sql/local/catalog_test_config_setup/`.
- `test/python/`: PySpark, PyIceberg, and stdin-driven DuckDB integration tests.
- `test/configs/`: unittest initialization and compatibility configs for the supported REST catalogs.
- `scripts/data_generators/`: pytest-driven Iceberg data generator and catalog connection profiles.
- `make/catalogs/`: local catalog lifecycle and data-generation targets.
- `extension_config.cmake`: DuckDB extensions included in the build.
- `CMakeLists.txt`: top-level extension dependencies and linking; per-directory `src/**/CMakeLists.txt` list the C++ sources for each directory (mirroring the DuckDB submodule layout).
- `duckdb/` and `extension-ci-tools/`: git submodules. Do not edit or update them unless the task explicitly requires it.

Keep existing user changes intact. Runtime directories such as `.catalogs/`, `.venv-spark4/`, `build/`, `data/generated/`, `.cache/`, and test temp directories are generated or ignored and should not be committed.

## Sources of truth

Read these before changing build or catalog behavior:

- The root `Makefile` defines extension settings and includes the catalog fragments.
- `extension-ci-tools/makefiles/duckdb_extension.Makefile` defines the standard build, test, format, clean, and update targets.
- `make/util.mk` owns `.catalogs/.active_catalog` and catalog switching.
- `make/catalogs/*.mk` owns service lifecycle and data generation.
- `scripts/data_generators/integration_config.py` owns REST catalog profiles, Spark runtimes, endpoints, and capabilities used by Python tests.
- `test/configs/*.json` owns unittest initialization SQL and catalog-specific skip policy.
- `.github/workflows/test-*-catalog.yml` shows the CI command sequence for each catalog.

Do not duplicate catalog credentials, endpoints, capabilities, or skip policy in a new location unless the task requires a new source of truth.

## Build and formatting

The build uses vcpkg. Set `VCPKG_TOOLCHAIN_PATH` to the vcpkg CMake toolchain unless the environment already provides it.

```shell
make debug
make release
make reldebug
make relassert
```

Build outputs live under `build/<configuration>/`. Standard test targets do not declare the corresponding build as a prerequisite, so build first.

```shell
make release && make test
make debug && make test_debug
make reldebug && make test_reldebug
```

Run formatting through the delegated targets:

```shell
make format-check
make format-fix
```

When adding a C++ implementation file, add it to the `OBJECT` library in the `CMakeLists.txt` of the directory the file lives in (each source directory has its own `CMakeLists.txt`, mirroring the DuckDB submodule layout). If you add a new directory, give it a `CMakeLists.txt` that appends its objects to `ALL_OBJECT_FILES` and `add_subdirectory()` it from its parent. Follow existing DuckDB C++ conventions and prefer the narrowest relevant build before a full rebuild.

## Testing expectations

Use the smallest relevant test while iterating, then broaden validation in proportion to the change.

Assume tests must run serially unless their isolation has been verified. Do not launch multiple unittest processes, pytest workers, CI shards, or agent-driven test commands in parallel merely to reduce runtime. Before introducing parallelism, inspect setup and cleanup and confirm that the tests do not share or mutate the same catalog, warehouse, namespace, table names, generated files, temporary paths, ports, or services. If isolation is uncertain, run the tests sequentially.

```shell
# A normal extension test
./build/debug/test/unittest test/sql/local/iceberg_scans/iceberg_scan.test

# A catalog-backed test
TEST_CONFIG="$(bash -c 'source scripts/catalog_test_config.sh && active_catalog_test_config')"
./build/debug/test/unittest \
  --test-config "$TEST_CONFIG" \
  test/sql/local/catalog_test_config_setup/catalog_agnostic/create/test_create_table.test

# Python integration tests
python3 -m pytest -vv test/python \
  --unittest-binary ./build/debug/test/unittest \
  --test-python-verbosity verbose
```

Sqllogictests use `.test`; expensive tests use `.test_slow`. Put behavior shared by all REST catalogs under `catalog_agnostic/` and catalog-only behavior under the existing `catalog_specifc/<catalog>/` tree. The directory name `catalog_specifc` is intentionally reproduced here because that is its current spelling; do not silently rename the tree.

Python tests outside `test/python/cloud/` require an active REST catalog. Cloud tests are only collected when requested explicitly. Use the existing `requires_spark`, `requires_capabilities`, and `spark_seed_tables` markers rather than open-coding environment checks.

## Catalog workflow and invariants

Supported REST catalog markers are `fixture`, `lakekeeper`, `nessie`, and `polaris`; `local` is valid for data generation but not for REST-catalog unittest config resolution.

```shell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckdb/duckdb-iceberg](https://github.com/duckdb/duckdb-iceberg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

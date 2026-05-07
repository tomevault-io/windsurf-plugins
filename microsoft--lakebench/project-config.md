---
trigger: always_on
description: > Quick-reference for Copilot and contributors. Keep this in sync when adding major features.
---

# LakeBench Codebase Reference

> Quick-reference for Copilot and contributors. Keep this in sync when adding major features.

---

## What is LakeBench?

LakeBench is a **Python-native, multi-modal benchmarking framework** for evaluating performance across multiple lakehouse compute engines and ELT scenarios. It supports industry-standard benchmarks (TPC-DS, TPC-H, ClickBench) and a novel ELT-focused benchmark (ELTBench), all installable via `pip`.

---

## Project Layout

```
src/lakebench/
├── __init__.py
│
├── benchmarks/
│   ├── base.py                  # BaseBenchmark ABC — result schema, timing, post_results()
│   ├── elt_bench/               # ELTBench: load, transform, merge, maintain, query
│   ├── tpcds/                   # TPC-DS: 99 queries, 24 tables
│   ├── tpch/                    # TPC-H: 22 queries, 8 tables
│   └── clickbench/              # ClickBench: 43 queries on clickstream data
│
├── datagen/
│   ├── tpch.py                  # TPCHDataGenerator (uses tpchgen-rs, ~10x faster than alternatives)
│   ├── tpcds.py                 # TPCDSDataGenerator (wraps DuckDB TPC-DS extension)
│   └── clickbench.py            # Downloads dataset from ClickHouse host
│
├── engines/
│   ├── base.py                  # BaseEngine ABC — fsspec, runtime detection, result writing
│   ├── spark.py                 # Generic Spark engine
│   ├── fabric_spark.py          # Microsoft Fabric Spark (auto-authenticates via notebookutils)
│   ├── synapse_spark.py         # Azure Synapse Spark
│   ├── hdi_spark.py             # HDInsight Spark
│   ├── duckdb.py                # DuckDB
│   ├── polars.py                # Polars
│   ├── daft.py                  # Daft
│   ├── sail.py                  # Sail (PySpark-compatible engine)
│   └── delta_rs.py              # Shared DeltaRs write helper (used by non-Spark engines)
│
└── utils/
    ├── query_utils.py           # transpile_and_qualify_query(), get_table_name_from_ddl()
    ├── path_utils.py            # abfss_to_https(), to_unix_path()
    └── timer.py                 # Context-manager timer; stores results for post_results()
```

---

## Core Abstractions

### `BaseEngine` (`engines/base.py`)
Abstract base for all compute engines.

| Attribute | Description |
|---|---|
| `SQLGLOT_DIALECT` | SQLGlot dialect string for auto-transpilation (e.g. `"duckdb"`) |
| `SUPPORTS_SCHEMA_PREP` | Whether the engine can create an empty schema-defined table before data load |
| `SUPPORTS_MOUNT_PATH` | Whether the engine can use mount-style URIs (`/mnt/...`) |
| `TABLE_FORMAT` | Always `'delta'` |
| `schema_or_working_directory_uri` | Base path where Delta tables are stored |
| `storage_options` | Dict passed through to DeltaRs / fsspec for cloud auth |
| `extended_engine_metadata` | Dict of key/value pairs appended to benchmark results |

Key methods: `get_total_cores()`, `get_compute_size()`, `get_job_cost(duration_ms)`, `create_schema_if_not_exists()`, `_append_results_to_delta()`.

Runtime is auto-detected at init via `_detect_runtime()` — returns `"fabric"`, `"synapse"`, `"databricks"`, `"colab"`, or `"local_unknown"`.

### `BaseBenchmark` (`benchmarks/base.py`)
Abstract base for all benchmarks.

| Attribute | Description |
|---|---|
| `BENCHMARK_IMPL_REGISTRY` | `Dict[EngineClass → ImplClass]` — maps engines to optional engine-specific implementations |
| `RESULT_SCHEMA` | Canonical 21-column result schema (see below) |
| `VERSION` | Benchmark version string |

The result schema includes: `run_id`, `run_datetime`, `lakebench_version`, `engine`, `engine_version`, `benchmark`, `benchmark_version`, `mode`, `scale_factor`, `scenario`, `total_cores`, `compute_size`, `phase`, `test_item`, `start_datetime`, `duration_ms`, `estimated_retail_job_cost`, `iteration`, `success`, `error_message`, `engine_properties` (MAP), `execution_telemetry` (MAP).

`post_results()` collects timer results → builds result rows → optionally appends to a Delta table via `engine._append_results_to_delta()`.

---

## Engine & Benchmark Registration

Benchmarks declare engine support via `BENCHMARK_IMPL_REGISTRY`. If an engine uses only shared `BaseEngine` methods, the value is `None`; otherwise it maps to a specialized implementation class.

```python
# Register a custom engine with an existing benchmark
from lakebench.benchmarks import TPCDS
TPCDS.register_engine(MyNewEngine, None)           # use shared methods
TPCDS.register_engine(MyNewEngine, MyTPCDSImpl)   # use custom impl class
```

To add a new engine, subclass an existing one:
```python
from lakebench.engines import BaseEngine

class MyEngine(BaseEngine):
    SQLGLOT_DIALECT = "duckdb"  # or whichever dialect applies
    ...

from lakebench.benchmarks.elt_bench import ELTBench
ELTBench.register_engine(MyEngine, None)
benchmark = ELTBench(engine=MyEngine(...), ...)
benchmark.run()
```

---

## Query Resolution Strategy (3-Tier Fallback)

For each query, LakeBench resolves in this order:

1. **Engine-specific override** — `resources/queries/<engine_name>/q14.sql` (rare; e.g. Daft decimal casting)
2. **Parent engine class override** — `resources/queries/<parent_class>/q14.sql` (rare; e.g. Spark family)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/LakeBench](https://github.com/microsoft/LakeBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

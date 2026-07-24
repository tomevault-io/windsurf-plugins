---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repo generates the Global Factor Data: 406 stock characteristics and their associated factor portfolios, based on "Is there a Replication Crisis in Finance?" by Jensen, Kelly, and Pedersen (Journal of Finance, 2023). It downloads data from WRDS (Wharton Research Data Services), computes characteristics from CRSP and Compustat sources, and constructs factor portfolios.

## Build & Run Commands

For complete setup and contribution instructions, see [CONTRIBUTING.md](CONTRIBUTING.md).

```bash
# Install dependencies
uv sync

# Run linting
uv run ruff check src/jkp/data/ tests/
uv run ruff format --check src/jkp/data/ tests/

# Run type checking (informational, not blocking in CI)
uv run pyright src/jkp/data/

# Run all unit tests
uv run pytest tests/unit/

# Run a specific test file or class
uv run pytest tests/unit/test_expressions.py
uv run pytest tests/unit/test_expressions.py::TestSumSas

# Run tests with coverage
uv run pytest

# Run the full pipeline (requires WRDS credentials and ~450 GB RAM)
jkp build data/                     # stock returns and firm characteristics
jkp portfolio data/                 # factor returns (run after jkp build)

# WRDS connection management
jkp connect                         # test connection
jkp connect --reset                 # reset stored credentials

```

## Architecture

The pipeline has two entry points that run sequentially:

**`src/jkp/data/main.py`** produces stock returns and firm characteristics:
1. Download raw data from WRDS (CRSP, Compustat)
2. Prepare and merge data sources (augmented monthly stock file, market cap/trading info)
3. Classify stocks by industry (Fama-French 49) and size (NYSE quintile cutoffs)
4. Compute characteristics from accounting and market data
5. Calculate rolling daily metrics (volatility, beta, skewness) across 21d/126d/252d/1260d windows
6. Save outputs as parquet files to `data/processed/`

**`src/jkp/data/portfolio.py`** constructs factor portfolios from the characteristics output by `main.py`, using ECDF ranking and value-weighting by market cap.

### Key source files

- `src/jkp/data/main.py` — Pipeline orchestration; calls functions from `aux_functions` in sequence
- `src/jkp/data/aux_functions.py` — Core library: all characteristic calculations, data transformations, and I/O utilities
- `src/jkp/data/portfolio.py` — Standalone factor portfolio construction script
- `src/jkp/data/wrds_credentials.py` — WRDS credential resolution (env vars, system keyring, and the libpq `~/.pgpass` file)

### Data flow

Raw WRDS data → `data/raw/` → intermediate processing in `data/interim/` → final outputs in `data/processed/` (subdirectories: `characteristics/`, `portfolios/`, `return_data/`, `accounting_data/`, `other_output/`).

Static reference data (`data/cluster_labels.csv`, `data/country_classification.xlsx`, `data/factor_details.xlsx`) is checked into the repo and used by the pipeline.

## Code Conventions

- **Polars, not Pandas.** The codebase uses Polars with lazy evaluation throughout. Always use `import polars as pl` and Polars APIs.
- **`pl.col()`, not bare `col()`.** Use `pl.col(...)` in new code. Existing code uses `from polars import col` with bare `col(...)`, but the standard Polars convention is the namespaced form.
- **Ruff:** Used for linting and formatting — abide by all rules specified in `pyproject.toml`
- **Python target:** See `requires-python` and `target-version` in `pyproject.toml`

## Development Guidelines

**Modularity & structure:**
- New pipeline functions should be added to `aux_functions.py` and called from `main.py`
- Use the `@measure_time` decorator on pipeline-level functions
- Use `collect_and_write()` for the lazy→eager→parquet workflow
- Pipeline functions in `aux_functions.py` that touch the filesystem must accept `paths: DataPaths` as their first parameter and use it for all path construction (e.g. `paths.interim_dir / "foo.parquet"`, `paths.raw_tables_dir / ...`). Never use `os.chdir`, `os.system`, or bare relative path strings. DuckDB SQL strings should interpolate `Path.as_posix()` rather than rely on cwd.

**Docstring format** (the codebase uses a consistent three-part format):
```
Description:
    What the function does.
Steps:
    1) ...
Output:
    What is written/returned.
```

**Polars patterns:**
- Use `pl.scan_parquet()` (lazy) for reading, not `pl.read_parquet()` unless full data is needed eagerly
- Return Polars expressions from helper functions (like `safe_div`)
- Use `safe_div()` for any division — never raw `/`
- **Do not use `sum_sas` or `sub_sas` in new code.** These are legacy functions that replicate SAS null semantics (treat null as 0 when any input is non-null). They exist for backward compatibility in existing characteristic calculations. New code should use standard Polars null propagation, or explicit `pl.coalesce()` when null-as-zero behavior is needed.
- Prefer Polars over DuckDB/Ibis for new code. Existing code uses DuckDB via Ibis for some complex SQL aggregations, but new work should use Polars expressions and lazy evaluation where possible.

**Type hints:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bkelly-lab/jkp-data](https://github.com/bkelly-lab/jkp-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

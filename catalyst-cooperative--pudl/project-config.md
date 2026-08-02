---
trigger: always_on
description: PUDL ingests raw public energy data (EIA, FERC, EPA, and others) and transforms it into
---

# AGENTS.md

## Project overview

PUDL ingests raw public energy data (EIA, FERC, EPA, and others) and transforms it into
clean, analysis-ready tables. The pipeline is orchestrated using Dagster assets and jobs.

## About this file

`AGENTS.md` is the canonical instruction file for this repository. `CLAUDE.md` in
the same directory is a symlink that points to `AGENTS.md` — they are always
identical, not independent files to be kept in sync.

### Working across multiple worktrees

When working in multiple git worktrees simultaneously, any `AGENTS.md` injected
into your context at session start reflects the **primary working directory** only.
If you encounter an `AGENTS.md` at a different path during the same session, do not
assume it is the same file — it may be a different version of this document, or
belong to an entirely different repository.

To avoid this confusion:

- Always use full absolute paths when referencing or comparing `AGENTS.md` files
  across worktrees — this makes the distinction visible immediately.
- When you need to know which `AGENTS.md` governs a particular worktree, read it
  directly from that worktree's directory rather than relying on the
  context-loaded version.

## Repository structure

Key directories under `src/pudl/`:

- `extract/` — one module per data source; reads raw inputs via the datastore and
  produces lightly-typed DataFrames
- `transform/` — one module per data source; cleans, normalizes, and validates data
- `dagster/` — all Dagster orchestration code; the canonical home for everything
  Dagster-specific. Sub-structure:
  - `dagster/asset_checks.py` — asset-check definitions, factories, and helpers
  - `dagster/assets/` — oddball asset definitions that don't fit the per-source layout;
    `assets/core/` for processed assets, `assets/raw/` for raw-extraction assets
  - `dagster/build.py` — assembles the `dagster.Definitions` object
  - `dagster/config.py` — reusable Dagster run-config fragments and helpers
  - `dagster/io_managers.py` — IO managers for SQLite, Parquet, and FERC SQLite reads
  - `dagster/jobs.py` — named Dagster jobs (`pudl`, `ferc_to_sqlite`, `ferceqr`)
  - `dagster/partitions.py` — shared partition definitions
  - `dagster/provenance.py` — FERC SQLite fingerprinting and compatibility checks
  - `dagster/resources.py` — `ConfigurableResource` definitions and default resource map
  - `dagster/sensors.py` — sensor-based automation (e.g. the FERC EQR sensor)
- `deploy/` — post-ETL deployment logic: publishing outputs to GCS/S3, updating
  `nightly`/`stable` git branches, triggering Zenodo releases, applying GCS holds,
  and redeploying the PUDL Viewer Cloud Run service. `ferceqr.py` contains Dagster
  assets specific to the FERC EQR batch pipeline; `pudl.py` covers full PUDL builds.
- `scripts/` — all CLI entry points as thin wrappers; one module per script, each
  exposing a `main` Click command. Registered in `[project.scripts]` in `pyproject.toml`
- `metadata/` — table and column metadata (`classes.py`, `fields.py`, `resources.py`);
  "Resources" are tables, "Fields" are columns
- `glue/` — entity resolution tables that link IDs across data sources
- `analysis/` — higher-level analytical assets built on top of the core ETL outputs
- `helpers.py` — shared utility functions; check here before writing new helpers
- `settings.py` — Pydantic settings models for all datasets and ETL configuration
- `validate.py` — data validation helpers that need to be accessible outside Dagster
  (foreign key checks, continuity checks)
- `definitions.py` — stable `dg`-compatible entry point; re-exports `defs` from
  `pudl.dagster`

Other important directories:

- `dbt/` — dbt models used for data validation only (not transformation)
- `tests/unit/` — fast unit tests; run these during development
- `tests/integration/` — slow integration tests; do not run interactively
- `docs/` — Sphinx documentation source (reStructuredText)
- `src/pudl/package_data/settings/` — packaged Dagster run config YAML files
  (`dg_fast.yml`, `dg_full.yml`, `dg_pytest.yml`, `dg_nightly.yml`)

## Development environment

### Inputs and outputs

Raw inputs and pipeline outputs live **outside the repository**, in directories with
sufficient disk space. Their locations are set by two environment variables:

- `$PUDL_INPUT` — root of the raw input datastore. Raw data files are downloaded here
  by the `pudl_datastore` CLI and read by the pipeline at runtime. Do not write to
  this directory manually.
- `$PUDL_OUTPUT` — root of all pipeline outputs. Contents include:
  - Apache Parquet files (`$PUDL_OUTPUT/parquet/`) — the primary analytical outputs
  - SQLite databases (`$PUDL_OUTPUT/*.sqlite`) — used for FERC raw data and some outputs
  - DuckDB databases (`$PUDL_OUTPUT/*.duckdb`) — currently only for FERC XBRL data
  - JSON datapackage descriptors (`$PUDL_OUTPUT/*_datapackage.json`) — frictionless
    datapackage metadata describing the schema and structure of the tabular outputs

Never assume these directories are inside the repository. Never hardcode paths to them.

### Python environment

PUDL uses `pixi` for dependency and task management. Always use `pixi run <command>` to
ensure commands run in the correct environment.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catalyst-cooperative/pudl](https://github.com/catalyst-cooperative/pudl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

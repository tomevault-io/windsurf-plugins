---
trigger: always_on
description: MEDS-DEV (Medical Event Data Standard — Decentralized Extensible Validation) is a **benchmark
---

# CLAUDE.md — MEDS-DEV

## What This Repo Is

MEDS-DEV (Medical Event Data Standard — Decentralized Extensible Validation) is a **benchmark
orchestration system**, not a model training framework. It stores YAML configuration files, task
definitions, training recipes, evaluation pipelines, and results — not runnable ML model code.
Models, datasets, and evaluation tools live in external packages; MEDS-DEV wraps them via CLI
entry points that create virtual environments, run commands, and collect outputs.

The core abstractions are:

- **Datasets** (`src/MEDS_DEV/datasets/`): Each has a `dataset.yaml` (ETL commands),
    `predicates.yaml` (ACES predicates), and `requirements.txt`.
- **Tasks** (`src/MEDS_DEV/tasks/`): ACES task configuration YAML files defining cohort
    extraction criteria. Tasks reference predicates that are overridden per-dataset.
- **Models** (`src/MEDS_DEV/models/`): Each has a `model.yaml` (train/predict shell commands
    with template variables), `requirements.txt`, and optional Python scripts.
- **Results** (`src/MEDS_DEV/results/`): JSON result packaging and validation.

## Build & Install

```bash
# Standard install (users reproducing benchmarks)
pip install MEDS-DEV

# Development install (contributors)
pip install -e ".[dev,tests]"
```

Requires Python ≥ 3.11. Uses `setuptools-scm` for versioning (version is derived from git tags).

## Running Tests

```bash
# Fast: doctests + unit tests only (no venvs, no data builds)
pytest --doctest-modules -m "not integration" -x

# Full suite (builds demo datasets, creates model venvs, runs end-to-end)
pytest -v -s -x

# Full suite with persistent caching (much faster on repeat runs)
mkdir -p /tmp/meds_dev_cache
pytest -v -s -x \
	--persistent_cache_dir=/tmp/meds_dev_cache \
	--cache_dataset=all --cache_task=all --cache_model=all \
	--reuse_cached_dataset=all

# Test a single model only
pytest -v -s -x --test_model=random_predictor

# Test a single task only
pytest -v -s -x --test_task=mortality/in_icu/first_24h
```

**Key test options:**

- `--persistent_cache_dir=DIR` — persist built datasets/models across runs (DIR must exist)
- `--cache_dataset=NAME|all` — cache dataset builds in persistent dir
- `--cache_task=NAME|all` — cache task extractions
- `--cache_model=NAME|all` — cache model outputs
- `--reuse_cached_dataset=NAME|all` — skip re-running dataset builds if cached
- `--test_dataset=NAME|all`, `--test_task=NAME|all`, `--test_model=NAME|all` — filter which
    components to test

Tests are **ordered** by `pytest_collection_modifyitems` in conftest.py to ensure datasets are
built before tasks, and tasks before models. Tests within a model are grouped so venvs can be
cleaned up between models.

## Code Quality

```bash
# Pre-commit runs the configured linters/formatters
pre-commit run --all-files
```

## Architecture

### Entry Points (CLI commands)

Defined in `pyproject.toml` under `[project.scripts]`:

- `meds-dev-dataset` → `MEDS_DEV.datasets.__main__:main`
- `meds-dev-task` → `MEDS_DEV.tasks.__main__:main`
- `meds-dev-model` → `MEDS_DEV.models.__main__:main`
- `meds-dev-evaluation` → `MEDS_DEV.evaluation.__main__:main`
- `meds-dev-pack-result` → `MEDS_DEV.results.__main__:pack_result`
- `meds-dev-validate-result` → `MEDS_DEV.results.__main__:validate_result`

All entry points use **Hydra** for configuration. Configs live in `src/MEDS_DEV/configs/` as
YAML files prefixed with `_`. Arguments are passed as Hydra overrides (dot-list syntax).

### Virtual Environment Isolation

Models and datasets run in isolated venvs to avoid dependency conflicts. The venv lifecycle is
managed by `src/MEDS_DEV/utils.py`:

- `install_venv(venv_dir, requirements)` — creates venv + pip installs requirements
- `temp_env(cfg, requirements)` — context manager that sets up venv + PATH
- `run_in_env(cmd, output_dir, env)` — runs a shell command in the venv, writes a `.done`
    sentinel on success to support idempotent reruns

### Registry Dictionaries

On import, `MEDS_DEV.__init__` populates three module-level dicts by scanning package resources:

- `DATASETS` — keyed by slash-separated path from `datasets/` dir (e.g., `"MIMIC-IV"`)
- `TASKS` — keyed by slash-separated path from `tasks/` dir (e.g., `"mortality/in_icu/first_24h"`)
- `MODELS` — keyed by slash-separated path from `models/` dir (e.g., `"meds_tab/tiny"`)

These are populated at import time via `importlib.resources.files()` + `rglob()`.

### Test Fixtures (tests/conftest.py)

The test suite uses **session-scoped parametrized fixtures** that chain together:

```
demo_dataset (builds MEDS data via meds-dev-dataset)
    → task_labels (extracts labels via meds-dev-task)
        → unsupervised_model (optional pretrain via meds-dev-model)
            → supervised_model (train + predict via meds-dev-model)
                → evaluated_model (evaluate via meds-dev-evaluation)
                    → packaged_result (pack via meds-dev-pack-result)
```

`pytest_generate_tests` dynamically parametrizes based on which tasks support which datasets
(via the `supported_datasets` field in task metadata). The cross-product of datasets × tasks ×
models is filtered by compatibility.

## Conventions

### Code Style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Medical-Event-Data-Standard/MEDS-DEV](https://github.com/Medical-Event-Data-Standard/MEDS-DEV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

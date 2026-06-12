---
trigger: always_on
description: Criteria Pattern is a typed Python package that standardizes criteria-based filtering, validation, ordering, pagination, and SQL/query conversion. It exposes value-object-backed models for filters, orders, and criteria, plus converters for SQL dialects and URL query parameters.
---

# AGENTS.md

## Project Overview

Criteria Pattern is a typed Python package that standardizes criteria-based filtering, validation, ordering, pagination, and SQL/query conversion. It exposes value-object-backed models for filters, orders, and criteria, plus converters for SQL dialects and URL query parameters.

Key paths:

- `criteria_pattern/models/criteria.py`: `Criteria` plus `AndCriteria`, `OrCriteria`, and `NotCriteria` composition.
- `criteria_pattern/models/filter/`: filter fields, operators, values, and the `Operator` enum.
- `criteria_pattern/models/order/`: order fields, directions, and the `Direction` enum.
- `criteria_pattern/models/filters.py` and `criteria_pattern/models/orders.py`: list value objects for filter and order collections.
- `criteria_pattern/converters/`: PostgreSQL, MySQL, MariaDB, SQLite, and URL-to-criteria converters.
- `criteria_pattern/errors/`: package-specific integrity, validation, and bounds errors.
- `criteria_pattern/models/testing/mothers/`: object mother helpers used by tests and downstream users.
- `tests/`: pytest suite organized to mirror `criteria_pattern/`.
- `pyproject.toml`: package metadata and tool configuration.
- `Makefile`: canonical local workflow.

This is a single-package Python project, not a monorepo. It supports Python `>=3.11` and CI tests Python `3.11`, `3.12`, `3.13`, and `3.14` on Ubuntu and Windows.

## Setup Commands

Run commands from the repository root.

- Show available project commands: `make help`
- Create the default virtual environment, install all dependency groups, and install hooks: `make setup`
- Create all configured virtual environments: `make setup-all`
- Install all dependency groups into an existing environment: `make install`
- Install one dependency group: `make install GROUP=test`, `make install GROUP=lint`, `make install GROUP=format`, or `make install GROUP=release`

The Makefile defaults to:

- `UV_BIN=uv`
- `PYTHON_VERSION=3.14`
- `PYTHON_VERSIONS=3.11,3.12,3.13,3.14`
- `PYTHON_VIRTUAL_ENVIRONMENT=.venv$(PYTHON_VERSION)`, so the default environment is `.venv3.14`

If Python 3.14 is unavailable locally, pass a supported version explicitly, for example:

```bash
make setup PYTHON_VERSION=3.13 PYTHON_VIRTUAL_ENVIRONMENT=.venv3.13
```

After setup, activate the environment when useful:

```bash
source .venv3.14/bin/activate
```

There is no database, message broker, or application server to start.

## Development Workflow

- Prefer the Make targets over ad hoc tool invocations.
- Keep changes scoped to the requested behavior; avoid unrelated cleanup.
- Add or update tests for behavior changes.
- For public API changes, update exports in `criteria_pattern/__init__.py` or package `__init__.py` files as needed.
- Keep `criteria_pattern/py.typed` present so package typing remains advertised.
- This repository currently has no lockfile; avoid introducing dependency lockfile churn unless the task is explicitly about dependency management.
- `make clean` removes configured virtual environments, caches, coverage files, and generated output. Treat it as destructive and do not run it unless the task calls for cleanup.

Use this local verification loop for code changes:

```bash
make format
make lint
make test
make coverage
```

For multi-version checks when the interpreters are available:

```bash
make test-all
make coverage-all
```

For documentation-only changes, run the smallest relevant check and state what was skipped if full verification is not needed.

## Testing Instructions

- Run all tests: `make test`
- Run all configured Python versions: `make test-all`
- Run coverage: `make coverage`
- Run all-version coverage: `make coverage-all`
- Run tests directly after setup: `.venv3.14/bin/python3.14 -m pytest --config-file pyproject.toml`
- Run a specific file: `.venv3.14/bin/python3.14 -m pytest tests/converters/test_criteria_to_postgresql_converter.py --config-file pyproject.toml`
- Run a focused test expression: `.venv3.14/bin/python3.14 -m pytest -k "url_to_criteria" --config-file pyproject.toml`
- Reproduce a randomized test order or data failure: `.venv3.14/bin/python3.14 -m pytest --config-file pyproject.toml --randomly-seed=<seed>`

If setup used a different `PYTHON_VERSION`, adjust the `.venv3.14/bin/python3.14` path or activate the environment and use `python -m pytest ...`.

Test conventions:

- Tests live under `tests/` and mirror package structure.
- Test files use `test_*.py` naming.
- Existing tests use `pytest.mark.unit_testing`.
- Assertions are plain `assert`; Ruff permits `assert` in test files.
- Test data helpers come from `object_mother_pattern` and `criteria_pattern.models.testing.mothers`.
- Tests use `pytest-randomly`, so do not write assertions whose expected value depends on random object-mother output ordering or primitive conversion side effects.
- Keep exact string and `to_primitives()` assertions deterministic. Prefer fixed values over object mothers when testing formatting or serialization output.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adriamontoto/criteria-pattern](https://github.com/adriamontoto/criteria-pattern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->

---
trigger: always_on
description: This file is for AI coding agents (Claude Code, Cursor, Codex, …) working on **dag-factory**, a
---

# AGENTS.md

This file is for AI coding agents (Claude Code, Cursor, Codex, …) working on **dag-factory**, a
Python library for [Apache Airflow®](https://airflow.apache.org/) that builds DAGs from YAML
configuration files.

## Environment Setup

- Install [`uv`](https://docs.astral.sh/uv/) and [`hatch`](https://hatch.pypa.io/) — `uv` manages the local virtualenv, `hatch` runs the test/docs matrix.
- Set up the dev environment with `uv sync --dev`. This creates `.venv/` with the right Python and all dependencies resolved from `pyproject.toml`. `make setup` is an alternative.
- Activate the venv: `source .venv/bin/activate` (or `source venv/bin/activate` if you used `make setup`).
- Install pre-commit hooks once: `pre-commit install`.
- To run example DAGs locally, export:
    - `AIRFLOW_HOME=$(pwd)/dev`
    - `AIRFLOW__CORE__LOAD_EXAMPLES=false`
    - `CONFIG_ROOT_DIR=$AIRFLOW_HOME/dags`

## Commands

| Task | Command |
| --- | --- |
| Unit tests (one matrix cell) | `hatch run tests.py3.10-2.9:test` |
| Unit tests with coverage | `hatch run tests.py3.10-2.9:test-cov` |
| Unit tests across the full matrix | `hatch run tests:test-cov` |
| Integration tests setup | `hatch run tests.py3.11-2.9:test-integration-setup` |
| Integration tests | `hatch run tests.py3.11-2.9:test-integration` |
| Static checks (ruff, black, codespell, …) | `pre-commit run --all-files` |
| Build wheel + sdist | `uv build --wheel --sdist` (or `make build-whl`) |
| Local Airflow via Astro CLI | `make docker-run` / `make docker-stop` |
| Docs (build + serve locally) | `hatch run docs:dev` |
| Docs (strict build) | `hatch run docs:build` |

Notes:

- The Airflow/Python matrix is in `pyproject.toml` under `[[tool.hatch.envs.tests.matrix]]`. Picking a `py<py>-<af>` cell that is not in the matrix will fail.
- Integration tests need `AIRFLOW_HOME`, `CONFIG_ROOT_DIR`, and `PYTHONPATH` pointing at `dev/` and `dev/dags`. See `docs/contributing/howto.md` for the full export block.
- Integration tests are selected by the `integration` pytest marker (`-m integration`); `tests/test_example_dags.py` is excluded from the unit run.
- `scripts/test/pre-install-airflow.sh` pulls Airflow constraints for the requested version — don't bypass it when reproducing CI failures locally.

## Repository Structure

```text
dag-factory/
├── dagfactory/         # Library source
│   ├── dagfactory.py   # Public entry points (load_yaml_dags)
│   ├── dagbuilder.py   # Translates YAML config into Airflow DAG/Task objects
│   ├── parsers.py      # Schedule/parameter parsing helpers
│   ├── _yaml.py        # YAML loading (safe loader, custom tags)
│   ├── constants.py    # Shared constants
│   ├── exceptions.py   # Library-specific exceptions
│   ├── settings.py     # Runtime settings / env var handling
│   ├── telemetry.py    # Anonymous usage telemetry (opt-out respected)
│   ├── utils.py        # Misc helpers
│   ├── listeners/      # Airflow listener integrations
│   └── plugin/         # Airflow plugin entry point (DagFactoryPlugin)
├── tests/              # Pytest suite, mirrors `dagfactory/`
│   ├── fixtures/       # YAML fixtures used by unit tests
│   └── fixtures_without_default_yaml/  # Fixtures for tests that omit a default YAML
├── dev/                # Local Astro/Airflow sandbox (Dockerfile, dags/, logs/)
│   └── dags/           # Example DAGs used locally and by tests/test_example_dags.py
├── examples/dags/      # Example YAML DAG configs included in the source tree
├── docs/               # mkdocs-material site
├── scripts/            # Test, doc, and release helpers
└── pyproject.toml      # Build + tool config (ruff, black, hatch, uv)
```

The library is single-package (`dagfactory`); there is no monorepo or workspace split. `dev/` and `examples/` are not packaged into the wheel (see `[tool.hatch.build.targets.wheel]`).

## Architecture Boundaries

dag-factory is a thin authoring layer that runs *inside* an Airflow deployment. Keep these responsibilities separate:

1. **YAML loading** (`_yaml.py`, `dagfactory.py`) reads config from disk or a Python dict and applies defaults from `defaults.yml`.
2. **DAG building** (`dagbuilder.py`) maps the parsed YAML onto Airflow primitives (DAG, Operator, TaskGroup, mapped tasks). Airflow-version compatibility shims live here — guard with `try/except ImportError` rather than version checks (see the `airflow.sdk.definitions.dag` fallback in `dagfactory.py`).
3. **Parsing helpers** (`parsers.py`) turn YAML strings into Airflow types (schedules, timedeltas, callbacks, Python callables).
4. **CLI** (`__main__.py`, the `dagfactory` Typer console script) is for operator commands; it should not import from runtime listener code.
5. **Telemetry** (`telemetry.py`) must stay opt-out and must never block DAG parsing if the network is down. Errors are swallowed by design.

Don't import Airflow at module top-level in code that may run before Airflow is initialized; prefer local imports or guarded `try/except ImportError`. dag-factory must keep working on both Airflow 2.9+ and Airflow 3.x.

## Security Model

Vulnerability reports go to `oss_security@astronomer.io` (see `SECURITY.md`). Don't file security issues on GitHub.

## Coding Standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [astronomer/dag-factory](https://github.com/astronomer/dag-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

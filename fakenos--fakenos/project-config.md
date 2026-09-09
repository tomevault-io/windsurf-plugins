---
trigger: always_on
description: This file gives Claude Code the key context needed to work productively in this repository.
---

# CLAUDE.md

This file gives Claude Code the key context needed to work productively in this repository.

## Project Overview

FakeNOS is a Python package for simulating network operating system CLI interactions. It starts fake SSH-accessible network devices and returns predefined or dynamically generated command output, making it useful for testing network automation without real routers, switches, or NOS VMs.

The project intentionally sits between unit-test mocks and full integration labs. It exercises connection establishment, SSH login, prompts, shells, and command responses, but it does not emulate network control, data, or management planes. Do not model it as a protocol emulator for BGP, LLDP, forwarding, routing, or vendor OS internals.

The public entry point is `FakeNOS`, exported from `fakenos/__init__.py` and implemented in `fakenos/core/fakenos.py`. The package also provides a `fakenos` test decorator and a `fakenos` CLI command via `fakenos.plugins.utils.cli:run_cli`.

Package metadata is in `pyproject.toml`:

- Package name: `fakenos`
- Current version: `1.1.0`
- Python support: `>=3.11,<3.15`
- Build backend: `uv_build`
- Core dependencies: `paramiko`, `pyyaml`, `pydantic`, `jinja2`, and `detect`
- Dev dependencies include `pytest`, `pytest-timeout`, `pytest-repeat`, `ruff`, `bandit`, `coverage`, `invoke`, `netmiko`, docs tooling, and YAML tooling

## Repository Layout

- `fakenos/core/`: core framework classes and validation.
  - `fakenos.py`: `FakeNOS`, default inventory, lifecycle methods, plugin registration, test decorator.
  - `host.py`: host object that wires together server, shell, and NOS plugins.
  - `nos.py`: base NOS plugin loader for YAML and Python plugin files.
  - `servers.py`: base TCP server abstraction.
  - `pydantic_models.py`: inventory, host, server, shell, and NOS validation models.
- `fakenos/plugins/`: built-in plugin systems.
  - `servers/ssh_server_paramiko.py`: Paramiko-backed SSH server plugin.
  - `shell/cmd_shell.py`: command shell implementation.
  - `nos/platforms_yaml/`: YAML platform definitions.
  - `nos/platforms_py/`: Python-backed dynamic platform definitions and templates.
- `tests/`: pytest suite split into `core` and `plugins`.
- `tests/assets/`: test inventories, test NOS modules, YAML fixtures, and SSH test keys.
- `docs/`: Zensical/MkDocs documentation source.
- `tasks.py`: Invoke tasks for linting, tests, docs, Docker image build, and Netmiko checks.
- `docker/`: Dockerfile and compose file for container-based FakeNOS testing.

## Development Setup

CI uses `uv`, so prefer it when available:

```bash
uv sync --all-groups
```

Then run commands through `uv run`, for example:

```bash
uv run pytest
```

This repository also contains a legacy `poetry.lock`, but the current CI, documentation, and build backend use `uv`.

## Running Tests

Run the whole pytest suite:

```bash
uv run pytest
```

If dependencies are already installed in the active environment, this also works:

```bash
python -m pytest
```

Run a specific test file:

```bash
uv run pytest tests/core/test_fakenos.py
```

Run a specific test:

```bash
uv run pytest tests/core/test_netmiko.py::TestNetmiko::test_testing_module
```

Run with coverage:

```bash
uv run coverage run -m pytest
uv run coverage report -m
uv run coverage html
```

Pytest configuration is in `pyproject.toml`; it sets `testpaths = ["tests"]` and `addopts = "-vv"`.

## Full Local Checks

The CI gates are Ruff, Bandit, and pytest. Run them locally with Invoke:

```bash
uv run invoke ruff --local
uv run invoke bandit --local
uv run invoke pytest --local
```

Or run the combined task:

```bash
uv run invoke tests --local
```

Important: Invoke tasks default to Docker execution unless `--local` is passed or `INVOKE_LOCAL=True` is set. On PowerShell:

```powershell
$env:INVOKE_LOCAL = "True"
uv run invoke tests
```

Without local mode, `invoke tests` expects the project Docker image to exist. Build it with:

```bash
uv run invoke build
```

The combined `tests` task currently runs Ruff, Bandit, and pytest. The YAML lint task exists but is commented out in `tasks.py` and disabled in CI.

## Invoke Tasks Reference

`tasks.py` defines project automation with Invoke. Run tasks as:

```bash
uv run invoke <task-name>
```

Most command-running tasks accept `--local`. Without `--local`, they run inside the Docker image named from `pyproject.toml` and tagged as `<version>-py<PYTHON_VER>`. Defaults are:

- `PYTHON_VER=3.14`
- `IMAGE_NAME=fakenos`
- `IMAGE_VER=1.1.0-py3.14`
- `INVOKE_LOCAL=False`

Environment variables can override these values.

Available tasks:

- `build`: builds the root `Dockerfile` image using `PYTHON_VER` as a build argument. Options: `--cache/--no-cache`, `--force-rm`, and `--hide`.
- `clean`: force-removes the project Docker image for the current `IMAGE_NAME:IMAGE_VER`.
- `rebuild`: runs `clean`, then rebuilds the Docker image without cache.
- `pytest`: runs `pytest`.
- `ruff`: runs `ruff check .`, then `ruff format --check`.
- `yamllint`: runs `yamllint .`. This exists, but it is not part of the combined `tests` task right now.
- `bandit`: runs `bandit -c pyproject.toml --recursive ./`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fakenos/fakenos](https://github.com/fakenos/fakenos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

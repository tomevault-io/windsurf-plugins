---
trigger: always_on
description: Official Python SDK for the Verda Cloud API (formerly DataCrunch). The main package is `verda`, with a backward-compatible `datacrunch_compat` wrapper that re-exports everything under the old `datacrunch` name.
---

# CLAUDE.md

## Project overview

Official Python SDK for the Verda Cloud API (formerly DataCrunch). The main package is `verda`, with a backward-compatible `datacrunch_compat` wrapper that re-exports everything under the old `datacrunch` name.

- **Package name:** `verda` (PyPI)
- **Python:** >= 3.10, tested on 3.10 through 3.14
- **License:** Apache 2.0
- **Build system:** `uv_build`

## Common commands

```bash
uv sync                          # install dependencies
uv run pytest                    # run all tests (unit + integration)
uv run pytest tests/unit_tests   # run unit tests only
uv run ruff check                # lint
uv run ruff format --check       # check formatting
uv run ruff format               # auto-format
```

CI runs three checks on every PR: unit tests (Python 3.10-3.14), `ruff check`, and `ruff format --check`.

## Project structure

```
verda/                  # main SDK source
  _verda.py             # VerdaClient entry point
  _version.py           # version string
  constants.py          # shared constants and enums
  exceptions.py         # APIException
  helpers.py            # utility functions
  http_client/          # HTTP wrapper with auth token refresh
  authentication/       # OAuth2 token management
  instances/            # instance lifecycle (create, delete, hibernate, etc.)
  volumes/              # storage volume management
  containers/           # serverless container deployments
  job_deployments/      # serverless batch jobs
  clusters/             # compute clusters
  images/               # OS images
  locations/            # datacenter locations
  ssh_keys/             # SSH key management
  startup_scripts/      # startup scripts
  balance/              # account balance
  inference_client/     # inference request handling
  instance_types/       # available instance types
  volume_types/         # available volume types
  cluster_types/        # available cluster types
  container_types/      # available container types
datacrunch_compat/      # backward-compat wrapper (deprecated)
tests/
  unit_tests/           # pytest unit tests, organized by module
  integration_tests/    # integration tests hitting real API
examples/               # usage examples
docs/                   # Sphinx documentation
.github/workflows/      # CI/CD (tests, lint, format, publish)
```

## Module conventions

Each service module follows this pattern:

```
verda/<service>/
  __init__.py           # public exports only (no license header)
  _<service>.py         # all implementation: dataclasses, service class, endpoint constant
```

- The `__init__.py` only re-exports public symbols from the private `_<service>.py` module.
- `__init__.py` files do NOT have the Apache 2.0 license header. All other `.py` files do.
- Implementation files are prefixed with `_` (e.g., `_instances.py`, `_volumes.py`).

## Code style

### Formatting and linting

- **Tool:** Ruff (lint + format)
- **Line length:** 100 characters
- **Quote style:** single quotes
- **Docstring convention:** Google style
- **Import sorting:** handled by ruff (`I` rules)

### Ruff rules enabled

`E4, E7, E9, F, C4, I, PT, B, UP, D, A, ARG, RUF`

Key rule groups: default pyflakes/pycodestyle, comprehensions, import sorting, pytest conventions, bugbear, Python syntax upgrades, pydocstyle (Google), builtins shadowing, unused args, ruff-specific.

Ignored: `F401` (unused imports), `B006` (mutable default args), `D100` (module docstring), `D105` (magic method docstring), `D107` (`__init__` docstring).

Per-file ignores:

- `tests/`, `examples/`, `datacrunch_compat/tests/` -- all `D` rules (docstrings not required)
- `__init__.py` -- `D104` (missing package docstring)
- `datacrunch_compat/datacrunch/*.py` -- `F403` (wildcard imports allowed)

### Type hints

- Use Python 3.10+ union syntax: `str | None` (not `Optional[str]`)
- Type-annotate function signatures and return types
- Use `Literal` for constrained string values (e.g., `Contract = Literal['LONG_TERM', 'PAY_AS_YOU_GO', 'SPOT']`)

### Dataclasses

- Use `@dataclass_json` decorator from `dataclasses_json` for JSON serialization
- Use `@dataclass_json(undefined=Undefined.EXCLUDE)` for API response models (tolerates extra fields)
- Stack decorators: `@dataclass_json` first, then `@dataclass`
- Optional fields use `field(default=None)` or `= None`
- Google-style docstrings with `Attributes:` section

### License header

All `.py` files (except `__init__.py`) must have this header at the top:

```python
# Copyright 2026 Verda Cloud Oy
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verda-cloud/sdk-python](https://github.com/verda-cloud/sdk-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

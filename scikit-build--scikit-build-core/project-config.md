---
trigger: always_on
description: handles:
---

# AGENTS.md

## Project

`scikit-build-core` is a PEP 517 build backend for CMake-based Python packages.
It is built with **hatchling** and stored under `src/scikit_build_core/`.

## Running things

- Prefer `uv run` over bare Python invocations. There is a `uv.lock` checked in.
- Use `uv sync` for a local dev install. `uv run` does this for you.
- `nox` is the task runner.
  - `nox -s tests` — run tests with xdist (`-n auto`).
  - `nox -s tests -- -k test_foo` — single test runner.
  - `nox -s docs` — serve the HTML docs in interactive mode.
  - `nox --non-interactive -s docs` — build the HTML docs without serving them.
  - `nox -t gen` — run all code generators (cog for README + schema + config
    reference + API docs).
  - `nox -s minimums` — lowest-direct dependency tests.
  - `nox -s pylint` - some slightly-slower lints not included in `prek`
- For linting, `prek -a --quiet` is preferred.

## Testing

- Tests live in `tests/`. Sample packages are in `tests/packages/` but **must
  not be recursed into** by pytest (`norecursedirs = ["tests/packages/**"]`).
- `tests/utils` is on `pythonpath` for test utilities.
- Many tests need a virtualenv (fixtures: `isolated`, `virtualenv`). These are
  auto-marked as `virtualenv` and `isolated` by `conftest.py`.
- Important pytest markers: `compile`, `configure`, `fortran`, `integration`,
  `isolated`, `network`, `setuptools`, `upstream`, `virtualenv`.
- `pytest -n auto` is the default parallelism. Some platforms retry on PyPy.

## Code quality

- **Ruff** handles linting and formatting. Do not import banned modules directly
  (e.g. use `scikit_build_core._compat.tomllib` instead of `tomli`/`tomllib`,
  `scikit_build_core._compat.typing.Self` instead of `typing.Self`). See
  `pyproject.toml` `tool.ruff.lint.flake8-tidy-imports.banned-api`.
- **mypy** is strict for `scikit_build_core.*`, less strict for tests. Config in
  `pyproject.toml`.
- **pylint** is sometimes run separately in CI (`nox -s pylint`).
- **prek**, a pre-commit rewrite in Rust, includes `check-sdist`,
  `validate-pyproject`, JSON schema checks, typos, shellcheck, and
  `sp-repo-review`.

## Things that bite

- **Banned imports are enforced by Ruff.** Use `scikit_build_core._compat.*`
  shims (`tomllib`, `typing.Self`, `importlib.metadata`, etc.) and
  `scikit_build_core._vendor.pyproject_metadata` — never the direct modules. See
  `tool.ruff.lint.flake8-tidy-imports.banned-api` in `pyproject.toml`.
- **`src/scikit_build_core/_vendor/` is vendored** (`pyproject_metadata`). Do
  not lint or hand-edit it.
- **Generated files** — editing the settings model or docstrings requires
  `nox -t gen`. This regenerates cog sections in `README.md` and
  `docs/reference/configs.md`, and
  `src/scikit_build_core/resources/scikit-build.schema.json`.
- **`tests/packages/`** are sample build fixtures and are excluded from pytest
  recursion (`norecursedirs`). `tests/utils` is on `pythonpath`.
- mypy is **strict** for `scikit_build_core.*`, relaxed for tests.

## Generated files

- `README.md` and `docs/reference/configs.md` contain cog-generated sections.
- `src/scikit_build_core/resources/scikit-build.schema.json` is generated from
  the Pydantic-style model.
- Run `nox -t gen` after changing any of these sources, then verify
  `git diff --exit-code` in CI.

## Architecture notes

`scikit-build-core` is the reference PEP 517 build backend for CMake-based
Python packages. Its job is to bridge the gap between Python's packaging
standards (`pyproject.toml`, SDists, wheels, editable installs) and CMake's
build system. The package is organized into a handful of coherent layers:

### High-level build flow

The entry points live in `src/scikit_build_core/build/__init__.py`. They
implement the standard PEP 517 hooks (`build_wheel`, `build_sdist`,
`build_editable`, `get_requires_for_build_*`, `prepare_metadata_for_build_*`).
The implementations are thin wrappers that:

1. Read `pyproject.toml` and construct a `SettingsReader`.
2. Run setup code (logging, validation).
3. Dispatch to the real workhorse, `_build_wheel_impl` in `build/wheel.py` or
   `build_sdist` in `build/sdist.py`.

For wheels/editable installs, the core flow is:

1. **Parse settings** (`settings/`): merge TOML, env vars, and PEP 517
   `config-settings` into a `ScikitBuildSettings` dataclass.
2. **Resolve metadata** (`build/metadata.py`): use the vendored
   `pyproject_metadata` library to turn `project` table data into
   `StandardMetadata`, including dynamic metadata plugins.
3. **Find CMake** (`cmake.py`, `program_search.py`): search the system (and
   optionally PyPI) for a CMake binary matching the requested version spec.
4. **Configure CMake** (`builder/builder.py`, `cmake.py`): set up the build
   directory, write an init-cache (`CMakeInit.txt`) with SKBUILD\_\* variables,
   and run `cmake -S … -B …`.
5. **Build** (`builder/builder.py`): run `cmake --build` with the correct
   generator (Ninja/Makefiles/MSVC).
6. **Install into wheel** (`builder/builder.py`): run `cmake --install` into a
   staging directory mapped to the wheel layout (`platlib`, `data`, `headers`,
   `scripts`, `metadata`).
7. **Package Python files** (`build/wheel.py`, `build/_pathutil.py`): copy
   discovered Python packages into the wheel, apply inclusion/exclusion rules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scikit-build/scikit-build-core](https://github.com/scikit-build/scikit-build-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

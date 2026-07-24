---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

InterpretML is a multi-language project. The repo holds several artifacts that build/ship together:

- `shared/libebm/` — C++11 native library `libebm` (the EBM training/scoring engine). Built into platform-specific `.dll` / `.so` / `.dylib` shared libraries placed in `bld/lib/`.
- `shared/vis/` — JavaScript/TypeScript visualization (`interpret-inline.js`) built with `npm` from `shared/vis/`.
- `python/interpret-core/` — main Python package (`interpret-core`). Contains the Python implementation of EBM, all explainers, the visual dashboard, and the ctypes bridge to `libebm`.
- `python/interpret/` — thin meta-package (`interpret`) that just depends on `interpret-core` plus extras.
- `python/powerlift/` — separate `powerlift` package for benchmarking (has its own pyproject and tests).
- `R/` — R package `interpret`, vendoring the same libebm sources.
- `docs/`, `scripts/`, `bld/` — documentation source, top-level Makefile/Docker orchestration, and build outputs.

Wheels do **not** build `libebm` from source — CI builds the shared library on each OS/arch and bundles the artifacts into the wheel. The C++ build is only triggered by an `sdist` install (see `python/interpret-core/setup.py` `BuildCommand` / `SDistCommand`) or by running `build.sh` / `build.bat` directly.

## Common commands

Working-directory cheat sheet (commands assume these cwds unless stated otherwise):
- Repo root — `./build.sh` / `./build.bat` and `clang-format-16` whole-tree (both reference `shared/libebm/` as a path argument; no `cd` into it).
- `python/` — `ruff check` and `ruff format` whole-tree (matches CI; keeps docs/JS/etc. out of scope; ruff still walks up to the repo-root `ruff.toml` for config).
- `python/interpret-core/` — Python install, `pytest`, `python -m mypy`.
- `shared/vis/` — npm commands for the visualization bundle.

### Native library (`libebm`)
- Linux/macOS: `./build.sh` (debug + release, default arch). Flags: `-release_64`, `-debug_64`, `-release_arm`, `-debug_arm`, `-release_32`, `-debug_32`, `-asm` (emit assembly), `-asan`, `-extra_debugging` (`-g`), `-conda` (use environment `CXX`/`CXXFLAGS`/`LDFLAGS`).
- Windows: `./build.bat` with the same `-release_64` / `-debug_64` / `-release_32` / `-debug_32` flags, plus `-analysis` for clang-tidy. Requires Visual Studio 2022 (`vcvars64.bat`).
- Output goes to `bld/lib/` (named `libebm[_<os>_<arch>][_debug].{dll,so,dylib}`).

### Python (run from `python/interpret-core/`)
- Install dev: `pip install -e ".[debug,notebook,plotly,lime,sensitivity,shap,linear,treeinterpreter,aplr,dash,skoperules,excel,testing]"`.
- Run all tests in parallel: `python -m pytest -vv -n auto`.
- Run a single test file: `python -m pytest -vv tests/glassbox/test_ebm.py`.
- Run a single test: `python -m pytest -vv tests/glassbox/test_ebm.py::test_name`.
- Pytest markers (defined in `pytest.ini`): `slow` and `selenium`. CI normally skips selenium; the Makefile passes `--runslow` (note: only honored if a corresponding conftest fixture is present). Filter with `-m "not slow"` or `-m "not selenium"` to skip.
- Coverage (matches CI): `python -m pytest -vv -n auto --cov=interpret --cov-report=xml`.

### JS visualization bundle (`shared/vis/`)
- Build (matches CI, Node 22): `cd shared/vis && npm install && npm run build-prod`. Output: `shared/vis/dist/interpret-inline.js`. The Python package re-bundles this file as package data, so Python users don't need npm unless they're editing the visualization.
- Dev variants: `npm run build-dev` (unminified), `npm start` (webpack-dev-server).

### Lint
- Whole tree: `cd python && ruff check` — scoped to `python/` (matches the same cwd convention as `ruff format`; ruff walks up to the repo-root `ruff.toml`). Single file: `ruff check path/to/file.py` from any cwd.
- Configuration is in `ruff.toml` (target `py310`, broad rule set including `B`, `I`, `PL`, `RUF`, `PERF`, `NPY`, `PD`, `UP`, etc.). `tests/**` waives `T20` (print). Project-wide Python floor is 3.10 — keep `ruff.toml`'s `target-version` and `pyproject.toml`'s `[tool.mypy] python_version` aligned if the floor is bumped.

### Type check (mypy)
- Config: `[tool.mypy]` in `python/interpret-core/pyproject.toml`.
- Run (matches CI): `cd python/interpret-core && python -m mypy`. CI runs with `continue-on-error: true`, so a failing mypy job won't break the build — but new strict modules should land clean.

### Format
After editing, format only the touched files. The whole-tree commands below are what CI runs.
- Python single file: `ruff format path/to/file.py` (works from any cwd; ruff walks up to `ruff.toml`/`pyproject.toml`). Check without rewriting: `ruff format --check path/to/file.py`. Whole tree (CI): `cd python && ruff format` — scoped to `python/` so docs and other non-package text aren't touched.
- C++ (always run in bash, even on Windows, so the versioned `clang-format-16` binary resolves and matches CI; `-style=file` picks up the repo's `.clang-format`):
  - Single file: `clang-format-16 -i -style=file path/to/file.cpp`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [interpretml/interpret](https://github.com/interpretml/interpret) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

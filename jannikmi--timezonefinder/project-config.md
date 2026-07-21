---
trigger: always_on
description: This Python library `timezonefinder` provides offline timezone lookups for WGS84 coordinates by combining preprocessed polygon data, H3-based spatial shortcuts, and optional acceleration via Numba or a clang-backed point-in-polygon routine. In comparison to other alternatives this package aims at maximum accuracy around timezone borders (no geometry simplifications) while offering fast lookup performance and compatibility with many (Python) runtime environments. The shipped dataset uses the full
---

# Agents Guide for timezonefinder

## Mission

This Python library `timezonefinder` provides offline timezone lookups for WGS84 coordinates by combining preprocessed polygon data, H3-based spatial shortcuts, and optional acceleration via Numba or a clang-backed point-in-polygon routine. In comparison to other alternatives this package aims at maximum accuracy around timezone borders (no geometry simplifications) while offering fast lookup performance and compatibility with many (Python) runtime environments. The shipped dataset uses the full original timezone dataset with all >440 timezone names, providing full localization capabilities and historical timezone accuracy.

## Repository Tour

- `timezonefinder/`: core library with `TimezoneFinder` (full polygon search), `TimezoneFinderL` (shortcut-only heuristic), global helper functions, CLI entry point, and utilities for polygon math and binary IO.
- `timezonefinder/data/`: packaged binary assets (FlatBuffers coordinate stores, NumPy metadata arrays, zone name list, shortcut index) consumed at runtime.
- `scripts/`: tooling for regenerating data (`file_converter.py`, `update_data.sh`), reporting, and helper configs shared by tests; relies on `uv` for builds.
- `tests/`: PyTest suite with fast unit coverage plus integration tests that build wheels/sdists inside venvs to validate packaging.
- `docs/`: Sphinx documentation mirroring PyPI content; `docs/data_format.rst` is the authoritative reference for binary layouts.
- `Makefile`, `tox.ini`, `pyproject.toml`: developer entry points for dependency sync, lint/test orchestration, and distribution metadata.
- `timezonefinder/command_line.py` defines the officially supported and tested CLI interface that mirrors the parts of the Python API.

## Runtime Model

The primary lookup flow converts query coordinates to scaled int32 values, collects candidate polygon IDs via the H3 shortcut map, rejects polygons whose bbox rules them out, checks holes first, and then applies a ray casting point-in-polygon test. When candidates share a timezone ID, the implementation short-circuits without extra geometry checks. Ocean zones (`Etc/GMT+/-XX`) guarantee a timezone match for all possible input coordinates unless callers explicitly use `timezone_at_land`.

## Data Pipeline

`update_data.sh` downloads a chosen timezone-boundary-builder release (`--dataset=full|same-since-now`, optional `--with-oceans`), unpacks it to `tmp/`, executes `scripts/file_converter.py` to emit FlatBuffers/NumPy assets under `timezonefinder/data/`, records the downloaded release tag in the `DATA_VERSION` file, bumps the patch version, prepends the matching `CHANGELOG.rst` entry, and deletes intermediates when `--rm-tmp` is passed. The script is non-interactive and safe to run in CI: `.github/workflows/check_data_updates.yml` compares `DATA_VERSION` against the latest upstream release weekly and, when a new release is available, runs the script and opens a ready-to-review update PR (falling back to a notification issue if the automated update fails). When the update PR's CI/CD pipeline passes, `.github/workflows/release_data_update.yml` merges it and pushes the version tag with a GitHub App token (the default `GITHUB_TOKEN` would not trigger the release pipeline in `build.yml`); on failure it labels the PR `automation-failed` and notifies the maintainer. The converter multiplies coordinates by 10^7, persists bboxes, hole registries, shortcut maps, and zone metadata; adjust `scripts/configs.py` when experimenting with alternative resolutions or debugging flags. When changing the datatype of shortcut-related FlatBuffers schemas (for example `hybrid_shortcuts_uint16.fbs`), delete any previously generated `.fbs` binary artifacts so they are regenerated consistently.

## Development Workflow

- useful commands are documented in the `Makefile`
- Install tooling via `uv sync --all-groups` (or `pip install timezonefinder[numba]` for runtime only); extras `numba` and `pytz` live in `pyproject.toml`.
- all python commands should be run via `uv run`
- whenever dependencies or the set of officially supported/tested Python versions change, update the lockfile with `make lock`
- **Testing**: Use make commands (`make test`, `make testint`, `make testall`) for global test runs. When only specific isolated unit tests are affected, run them directly via `uv run pytest tests/path/to/test_file.py::test_name` or `uv run pytest -k "test_pattern"`. For full test suites: `make test` (unit tests excluding integration and slow), `make testint` (integration tests), `make testall` (all tests including slow), or `uv run tox` (all environments).
- Format/lint: Ruff, isort, mypy, and pre-commit hooks are wired through `pyproject.toml` and the `Makefile` targets (`make hook`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jannikmi/timezonefinder](https://github.com/jannikmi/timezonefinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

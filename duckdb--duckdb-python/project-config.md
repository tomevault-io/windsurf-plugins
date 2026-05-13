---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project overview

This is the **production** duckdb-python client — the `duckdb` package on PyPI. It provides Python bindings for [DuckDB](https://duckdb.org), an in-process OLAP database engine, via pybind11 and a custom scikit-build-core build backend.

- **Repository**: https://github.com/duckdb/duckdb-python
- **Package name**: `duckdb`
- **Bindings**: pybind11
- **Build backend**: `duckdb_packaging.build_backend` (custom wrapper around scikit-build-core)
- **Supported Python**: 3.10, 3.11, 3.12, 3.13, 3.14
- **Free-threaded Python**: not supported in this client. A separate prototype client based on DuckDB's C API targets free-threading, Stable ABI, and multi-interpreter support.

## IMPORTANT: build before running anything

**You MUST complete a full build before running tests, scripts, or `uv run` in a fresh worktree or after a clean slate.** `uv run pytest` triggers scikit-build-core's editable rebuild on import, which compiles 2000+ C++ files from scratch — this takes 5–10 minutes and will exceed the Bash tool's default timeout. Do not attempt to run tests, scripts, or `uv run python` until the two-step build below has completed successfully.

```bash
# Step 1: install build deps (~5 seconds)
uv sync --only-group build --no-install-project -p 3.13

# Step 2: build the extension (3–10 min cold, ~30s with sccache, use timeout: 600000)
uv sync --no-build-isolation -v --reinstall -p 3.13
```

After step 2 completes, `uv run pytest`, `uv run python`, and `.venv/bin/python` all work immediately. Subsequent C++ changes trigger fast incremental rebuilds (seconds with sccache), not full cold builds.

## Build system

### Editable install (standard development workflow)

The build uses `--no-build-isolation` for two reasons: (1) it makes incremental rebuilds fast by reusing the target venv, and (2) **it keeps pybind11 headers in the venv so CLion / other C++ IDEs can resolve them** — with build isolation, pybind11 is installed into a temp env that gets destroyed after the build, leaving `compile_commands.json` with stale include paths and CLion unable to navigate the bindings code.

`--no-build-isolation` requires that build dependencies are already installed. **This is a two-step process on a fresh venv:**

**Step 1 — install build deps** (fast, ~5 seconds):

```bash
uv sync --only-group build --no-install-project -p 3.13
```

**Step 2 — build the extension** (3–10 min cold, ~30 seconds with warm sccache):

```bash
uv sync --no-build-isolation -v --reinstall -p 3.13
```

This produces a debug editable install in `build/debug/` with `editable.mode = "redirect"`. Python code changes are picked up immediately; C++ changes require a rebuild.

### sccache (strongly recommended)

sccache caches compiled object files across builds and worktrees. Without it, cold builds take 5–10 minutes (2000+ C++ compilation units). With a warm cache, they take ~30 seconds.

```bash
# Export before any uv sync
export CMAKE_C_COMPILER_LAUNCHER="$(command -v sccache)"
export CMAKE_CXX_COMPILER_LAUNCHER="$(command -v sccache)"
```

Install with `brew install sccache` (macOS). Check cache state with `sccache -s`.

### Non-editable (release-style) install

```bash
uv sync --no-build-isolation --no-editable -v --reinstall -p 3.13
```

Produces a release build (no debug symbols, optimized).

### Wheel build

```bash
uv build --wheel
```

Produces a wheel in `dist/`. Uses cibuildwheel for CI — see `pyproject.toml` `[tool.cibuildwheel]` for the CI wheel matrix (macOS arm64/x86_64, Linux x86_64/aarch64, Windows AMD64/ARM64).

### sdist build

```bash
uv build --sdist
```

The sdist includes the DuckDB submodule source via the `[tool.scikit-build.sdist]` include list in `pyproject.toml`.

### Incremental rebuild shortcuts

After editing C++ source:

```bash
# Fastest: touch the __init__.py to trigger scikit-build-core's rebuild detection
touch duckdb/__init__.py && uv sync --no-build-isolation -v --reinstall
```

Rebuild only the duckdb package (useful when dependency lock hasn't changed):

```bash
uv sync --reinstall-package duckdb
```

### Clean slate

```bash
rm -rf build .venv uv.lock && uv cache clean --force
```

### Python version selection

Pass `-p <version>` to any `uv sync` command:

```bash
uv sync --no-build-isolation -v --reinstall -p 3.11
uv sync --no-build-isolation -v --reinstall -p 3.14
```

Supported: `3.10`, `3.11`, `3.12`, `3.13`, `3.14`. Do **not** use free-threaded variants (`3.13t`, `3.14t`) — the production client does not support them.

### Build configuration reference

Key `pyproject.toml` settings:

- `BUILD_EXTENSIONS = "core_functions;json;parquet;icu;jemalloc"` — extensions built into the wheel.
- Editable overrides: `build-dir = "build/debug/"`, `editable.rebuild = true`, `editable.mode = "redirect"`, `cmake.build-type = "Debug"`, `DISABLE_UNITY = "1"` (unity disabled for better debugging).
- Coverage overrides: `build-dir = "build/coverage/"`, `RelWithDebInfo`, `--coverage` flags. Activate with `COVERAGE=true uv sync ...`.

## Testing

### Test layout

```
tests/
├── fast/              # quick per-subsystem tests (seconds each)
│   ├── arrow/         # pyarrow integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckdb/duckdb-python](https://github.com/duckdb/duckdb-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

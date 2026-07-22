---
trigger: always_on
description: handles this by passing `-DCMAKE_INSTALL_LIBDIR=lib` to cmake. If you modify
---

# Agent Development Guide

This repository wraps the C++ `LibRaw` library using Cython.

## Prerequisites

Before starting, ensure you have:
- **Python 3.9+**
- **C++ compiler** - `apt install g++` (Ubuntu) / Xcode Command Line Tools (macOS)

## Critical: Compilation Required

**You are working with Cython (`.pyx`) files.**
Changes to `rawpy/_rawpy.pyx` or C++ files **will not take effect** until you recompile.

| File type | After editing... |
|-----------|------------------|
| `.py` files | Changes apply immediately (editable install) |
| `.pyx` files | Must run `bash scripts/rebuild.sh` |
| C++ files in `external/` | Must run `bash scripts/rebuild.sh` |
| `MANIFEST.in` | Rebuild: `bash scripts/build_dist.sh` |

## Quick Commands

| Task | Command |
|------|---------|
| First-time setup | `bash scripts/setup_dev_env.sh` |
| Setup with specific Python | `bash scripts/setup_dev_env.sh 3.12` |
| Activate environment | `source .venv/bin/activate` |
| Rebuild after .pyx/C++ changes | `bash scripts/rebuild.sh` |
| Quick sanity check | `bash scripts/dev_check.sh` |
| Build sdist + wheel | `bash scripts/build_dist.sh` |
| Test built sdist | `bash scripts/test_dist.sh sdist` |
| Test built wheel | `bash scripts/test_dist.sh wheel` |
| Test with numpy version | `bash scripts/test_dist.sh wheel 2.0.2` |
| Test sdist with system libraw | `bash scripts/build_dist.sh && RAWPY_USE_SYSTEM_LIBRAW=1 bash scripts/test_dist.sh sdist` |
| Test wheel with system libraw | `RAWPY_USE_SYSTEM_LIBRAW=1 bash scripts/build_dist.sh && RAWPY_USE_SYSTEM_LIBRAW=1 bash scripts/test_dist.sh wheel` |
| Run single test | `pytest test/test_basic.py::testName -v` |
| Run all tests | `pytest test/` |
| Type check | `mypy rawpy` |
| Switch numpy version | `bash scripts/setup_numpy.sh 2.0.2` |
| Build docs | `cd docs && sphinx-build -b html . _build/html` |
| Serve & view docs | `cd docs/_build/html && python -m http.server 8765` then open `http://localhost:8765` |

> **System libraw requires LibRaw ≥ 0.21.** Ubuntu 22.04's `libraw-dev` (0.20.2) is
> too old. Use Ubuntu 24.04+ or build without `RAWPY_USE_SYSTEM_LIBRAW`.
>
> Note: The sdist build command does **not** use `RAWPY_USE_SYSTEM_LIBRAW=1`
> because sdist just packages source files — it doesn't compile anything. The
> env var is only needed at install/test time, when pip builds the sdist from
> source. For wheel, the env var is needed at both build **and** test time.

## Environment Setup

**First time only:**
```bash
bash scripts/setup_dev_env.sh
```

This will:
1. Create a `.venv` virtual environment
2. Check for required system dependencies (cmake, C++ compiler)
3. Initialize git submodules (LibRaw source)
4. Install Python dependencies
5. Build and install rawpy in editable mode

**With a specific Python version (Ubuntu only):**
```bash
bash scripts/setup_dev_env.sh 3.12
```

This installs the requested Python via the deadsnakes PPA, creates a `.venv`
with it, then runs the full setup. You can also use `scripts/setup_python.sh`
directly if you only need to switch the Python version without rebuilding.

**For subsequent sessions:**
```bash
source .venv/bin/activate
```

## Architecture

| Path | Purpose |
|------|---------|
| `rawpy/_rawpy.pyx` | Main Cython implementation (RawPy class, C++ bindings) |
| `rawpy/_rawpy.cpp` | **Generated** C++ from `.pyx` — do not edit manually. `setup.py` calls `cythonize()` which regenerates this, but only when the `.pyx` has a newer timestamp than the `.cpp`. A stale `.cpp` from a previous build can cause failures if the NumPy ABI has changed. `scripts/rebuild.sh` deletes it to force regeneration. |
| `rawpy/_rawpy.pyi` | Type stubs (update when changing API) |
| `rawpy/__init__.py` | Python entry point |
| `rawpy/enhance.py` | Pure Python utilities (bad pixel repair, etc.) |
| `external/LibRaw/` | LibRaw C++ library (git submodule) |
| `external/LibRaw/libraw/*.h` | LibRaw headers (check these for C++ signatures) |
| `external/LibRaw-cmake/` | CMake build system for LibRaw (git submodule) |
| `setup.py` | Build configuration (compiles LibRaw from source, links Cython extension) |
| `tmp/` | Scratch directory for build logs etc. (git-ignored) |
| `.github/workflows/ci.yml` | CI workflow (build matrix for Linux/macOS/Windows × Python versions) |
| `.github/scripts/` | Platform-specific CI build/test scripts |

## Common Tasks

### Building and viewing documentation

The docs use Sphinx with the Read the Docs theme. Both are already installed
in the dev venv (via `dev-requirements.txt`).

1. Build: `cd docs && sphinx-build -b html . _build/html`
2. Serve: `cd docs/_build/html && python -m http.server 8765` (run as background process)
3. Open `http://localhost:8765` in the Simple Browser

- Source files: `docs/index.rst`, `docs/api/*.rst`
- Config: `docs/conf.py`
- Output: `docs/_build/html/` (git-ignored)
- The docs use `autodoc` to pull docstrings from the built Cython extension,
  so `rawpy._rawpy` must be importable (i.e., the extension must be compiled).
  Run `bash scripts/rebuild.sh` first if needed.

### Adding a new LibRaw method

1. Find the C++ signature in `external/LibRaw/libraw/libraw.h`
2. Add the `cdef extern` declaration in `rawpy/_rawpy.pyx`
3. Add a Python method in the `RawPy` class in `rawpy/_rawpy.pyx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [letmaik/rawpy](https://github.com/letmaik/rawpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

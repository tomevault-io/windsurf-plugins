---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Aydin is a self-supervised, auto-tuned image denoising tool for n-dimensional images. It provides four interfaces: GUI (Aydin Studio), napari plugin, CLI, and Python API. Supports classical denoisers, patch-based methods, and machine learning approaches (CNN, Gradient Boosting).

## Common Commands

### Development Setup
```bash
pip install -e ".[dev]"
# Or use the Makefile (also installs docs deps + pre-commit hooks):
make setup
```

### Testing
```bash
# Run all tests (excludes heavy, gpu, unstable by default)
make test

# Run a single test file
pytest src/aydin/path/to/test_file.py --disable-pytest-warnings

# Run a single test function
pytest src/aydin/path/to/test_file.py::test_function_name --disable-pytest-warnings

# Run heavy tests only (marked @pytest.mark.heavy)
make test-heavy

# Run GPU tests only (marked @pytest.mark.gpu)
make test-gpu
```

### Code Style
```bash
# Format code (isort + black)
make format

# Check formatting and lint
make check
```

### Running the Application
```bash
aydin                       # Launch GUI (Aydin Studio)
aydin denoise image.tif     # CLI denoising
aydin -h                    # Help
```

### Documentation
```bash
make docs                   # Build HTML docs (current version)
make docs-build             # Build multi-version docs (all tags)
make docs-publish           # Build and deploy to GitHub Pages
```

## Architecture

### Project Layout

Uses a **`src/` layout**: source code lives in `src/aydin/`. This prevents accidental imports of the uninstalled package from the repo root.

### Core Components

**Image Translator Framework** (`src/aydin/it/`):
- `base.py` - Core `ImageTranslatorBase` class that handles multi-dimensional arrays with batch/channel/spatial dimensions, training/inference slicing, transforms, and normalization
- `classic.py`, `cnn_torch.py`, `fgr.py` - Implementations wrapping different denoising approaches

**Denoising Algorithms** (`src/aydin/restoration/denoise/`):
- `classic.py` - Classical denoisers (Butterworth, Gaussian, NLM, Total Variation)
- `noise2selffgr.py` - Self-supervised Feature Generation & Regression (recommended approach)
- `noise2selfcnn.py` - Self-supervised CNN approach

**Regression Methods** (`src/aydin/regression/`):
- `cb.py` - CatBoost (default for FGR)
- `lgbm.py` - LightGBM
- Other: linear, perceptron, random forest, SVM

**Feature Engineering** (`src/aydin/features/`):
- `standard_features.py`, `extensible_features.py` - Feature generation for ML denoisers

**Transforms** (`src/aydin/it/transforms/`):
- Image preprocessing/postprocessing: deskew, motion correction, high-pass, range normalization, histogram stretching, VST, padding

**I/O** (`src/aydin/io/`):
- `io.py` - Image reading/writing supporting TIFF, CZI, ND2, Zarr formats

**Interfaces**:
- `src/aydin/gui/` - PyQt6-based GUI (Aydin Studio)
- `src/aydin/cli/cli.py` - Click-based CLI

### Build System

- **Build backend**: hatchling (configured in `pyproject.toml`)
- **Version**: Single source of truth is `src/aydin/__init__.py` (`__version__`). `pyproject.toml` reads it dynamically via `[tool.hatch.version]`.
- **Makefile**: Orchestrates common commands (`make help` for full list)
  - `make setup` / `make install-dev` - Install for development
  - `make test` / `make test-heavy` / `make test-gpu` - Run tests
  - `make format` / `make check` - Code formatting and linting
  - `make validate` - Pre-publish checks (format + lint + clean tree)
  - `make build` / `make clean` - Build and clean artifacts
  - `make publish` / `make publish-patch` - Create release PR (see Release Process below)

### Test Markers

Tests use pytest markers defined in `pyproject.toml`:
- `@pytest.mark.heavy` - Long-running tests
- `@pytest.mark.gpu` - Requires NVIDIA GPU
- `@pytest.mark.unstable` - Flaky tests

### Logging

Use the internal logging API at `src/aydin/util/log/log.py` instead of print statements.

### Versioning

The project uses **calendar versioning**: `YYYY.M.D` (e.g., `2025.2.4`), with an optional `.patch` suffix for same-day releases (e.g., `2025.2.4.1`).

- **Single source of truth**: `src/aydin/__init__.py` → `__version__ = "YYYY.M.D"`
- `pyproject.toml` reads it dynamically via `[tool.hatch.version]` — never edit the version there
- `docs/source/conf.py` also reads from `__init__.py` at build time

### Release Process

Releases go through a PR-based flow. Requires the GitHub CLI (`gh`).

```bash
# Regular release (bumps to today's date, e.g., 2025.2.16)
make publish

# Patch release (e.g., 2025.2.16.1 → 2025.2.16.2)
make publish-patch
```

**What `make publish` does:**
1. Runs `make validate` (checks: on main, clean tree, formatting, lint)
2. Creates a `release/vYYYY.M.D` branch
3. Bumps `__version__` in `src/aydin/__init__.py`
4. Pushes and creates a PR via `gh pr create`
5. Switches back to `main`

**After that, you merge the PR on GitHub.** Then the automated pipeline takes over:

```
PR merged → release.yml creates git tag → publish.yml: verify → test → build → PyPI → GitHub Release
```

### CI/CD Pipeline

**Workflows** (`.github/workflows/`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [royerlab/aydin](https://github.com/royerlab/aydin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

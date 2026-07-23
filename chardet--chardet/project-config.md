---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

0BSD-licensed, ground-up rewrite of chardet (the Python character encoding detector). Drop-in replacement for chardet 6.x — same package name, same public API. Zero runtime dependencies, Python 3.10+, must work on PyPy.

### Versioning

Version is derived from git tags via `hatch-vcs`. The tag is the single source of truth — no hardcoded version strings. Tags do **not** use a `v` prefix: the tag for version 7.3.0 is `7.3.0`, not `v7.3.0`. At tag `7.0.0` the version is `7.0.0`; between tags it's auto-incremented (e.g., `7.0.1.dev3+g...`). The generated `src/chardet/_version.py` is gitignored and should never be committed.

When releasing `X.Y.Z`, also tag the `chardet/test-data` repo with `X.Y.Z` at its current `main` HEAD. The accuracy test suite clones test-data at the matching version tag for release builds, falling back to `main` for dev builds. This ensures `test_accuracy.py` continues to pass for released versions even after test-data is updated.

## Commands

### Development Setup

```bash
uv sync                    # install dependencies
prek install               # set up pre-commit hooks (ruff lint+format, trailing whitespace, etc.)
```

### Testing

```bash
uv run python -m pytest -n auto                      # run all tests (excludes benchmarks)
uv run python -m pytest -n auto tests/test_api.py    # run a specific test file
uv run python -m pytest tests/test_api.py::test_detect_empty  # run a single test
uv run python -m pytest -m benchmark -n auto         # run benchmark tests only
uv run python -m pytest -x -n auto                   # stop on first failure
```

Test data is auto-cloned from `chardet/test-data` GitHub repo on first run (cached in `tests/data/`, gitignored). Accuracy tests are dynamically parametrized from this data via `conftest.py`.

### Linting & Formatting

```bash
uv run ruff check .        # lint
uv run ruff check --fix .  # lint with auto-fix
uv run ruff format .       # format
```

### Training Models

```bash
uv run python scripts/train.py   # retrain bigram models from CulturaX/MADLAD-400/Wikipedia data
uv run python scripts/verify_no_overlap.py  # verify no train/test data overlap
```

Training data is cached in `data/` (gitignored) under `data/culturax/`,
`data/madlad400/`, and `data/wikipedia/` per language. Models are saved to
`src/chardet/models/models.bin`. Test data articles are automatically excluded
from training via content fingerprinting to prevent train/test overlap.

### Benchmarks & Diagnostics

```bash
uv run python scripts/benchmark_time.py     # latency benchmarks
uv run python scripts/benchmark_memory.py   # memory usage benchmarks
uv run python scripts/diagnose_accuracy.py  # detailed accuracy diagnostics
uv run python scripts/compare_detectors.py  # compare against original chardet
```

### Documentation

```bash
uv sync --group docs                          # install Sphinx, Furo, etc.
uv run sphinx-build docs docs/_build          # build HTML docs
uv run sphinx-build -W docs docs/_build       # build with warnings as errors
uv run python scripts/generate_encoding_table.py > docs/supported-encodings.rst  # regenerate encoding table
```

Docs use Sphinx with Furo theme. API reference is auto-generated from source docstrings via autodoc. Published to ReadTheDocs on tag push (`.readthedocs.yaml`). Source files are in `docs/`; `docs/plans/` is excluded from the build.

### Building with mypyc (optional)

```bash
HATCH_BUILD_HOOK_ENABLE_MYPYC=true uv build  # compile hot-path modules
```

Compiled modules: `models/__init__.py`, `pipeline/structural.py`, `pipeline/validity.py`, `pipeline/statistical.py`, `pipeline/utf1632.py`, `pipeline/utf8.py`, `pipeline/escape.py`, `pipeline/orchestrator.py`, `pipeline/confusion.py`, `pipeline/magic.py`, `pipeline/ascii.py`, `pipeline/language.py`, `pipeline/postprocess.py`. These modules cannot use `from __future__ import annotations` (FA100 is ignored for them in ruff config).

## Architecture

### Detection Pipeline (`src/chardet/pipeline/orchestrator.py`)

All detection flows through `run_pipeline()`, which runs stages in order — each stage either returns a definitive result or passes to the next:

1. **BOM** (`bom.py`) — byte order mark → confidence 1.0
2. **UTF-16/32 patterns** (`utf1632.py`) — null-byte patterns for BOM-less Unicode
3. **Escape sequences** (`escape.py`) — ISO-2022-JP/KR, HZ-GB-2312
4. **Magic numbers** (`magic.py`) — file signature detection (e.g., PDF, images)
5. **Binary detection** (`binary.py`) — null bytes / control chars → encoding=None
6. **Markup charset** (`markup.py`) — `<meta charset>` / `<?xml encoding>` extraction, plus `promote_markup_superset()` which swaps the declared encoding to its Windows superset (Shift_JIS→CP932, EUC-KR→CP949) when structural evidence supports it
7. **ASCII** (`ascii.py`) — pure 7-bit check
8. **UTF-8** (`utf8.py`) — structural multi-byte validation
9. **Byte validity** (`validity.py`) — eliminate encodings that can't decode the data
10. **CJK gating** (in orchestrator) — eliminate CJK candidates lacking multi-byte structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chardet/chardet](https://github.com/chardet/chardet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

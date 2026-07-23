---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ProteoBench is a community-curated benchmarking platform for proteomics data analysis pipelines. It compares outputs from mass spectrometry tools (MaxQuant, DIA-NN, FragPipe, Spectronaut, AlphaDIA, Sage, PEAKS, MSAID, AlphaPept, i2MassChroQ, ProlineStudio, MSAngel, WOMBAT, MetaMorpheus, quantms, Proteome Discoverer) across different acquisition modes (DDA/DIA) and instrument types (QExactive, Astral, diaPASEF, AIF, ZenoTOF, low input). It also hosts a de novo sequencing module (Casanovo, InstaNovo, and related tools).

The core library (`proteobench/`) processes tool outputs into a standardized format and computes quantification metrics. A Streamlit web app (`webinterface/`) provides the interactive UI. Benchmark results are stored as individual JSON files and submitted via GitHub pull requests to separate results repositories (e.g., `Proteobench/Results_quant_ion_DDA`).

## Common Commands

```bash
# Install for development
pip install -e '.[dev]'

# Optional extras: docs tooling, or the web app (streamlit>1.27, scipy)
pip install -e '.[docs]'
pip install -e '.[web]'

# Run all tests
pytest

# Run a single test file
pytest test/test_parse_params_maxquant.py

# Run a single test by name
pytest test/test_module_quant_ion_DDA_QExactive.py -k "test_benchmarking_return_types"

# Run tests via nox (uses uv backend)
nox --session "tests"

# Run notebooks via nox
nox --session "test_notebooks"

# Check formatting (CI uses black ~=23.0)
black --check proteobench

# Fix formatting
black proteobench

# Lint (CI only fails on E9,F63,F7,F82)
flake8 . --select=E9,F63,F7,F82

# Start the web interface
cd webinterface && streamlit run Home.py

# Build docs with live preview
pip install -e '.[docs]'
sphinx-autobuild --watch ./proteobench ./docs/ ./docs/_build/html/

# Build docs via nox (supports --serve and -b linkcheck)
nox --session "docs"
nox --session "docs" -- --serve
nox --session "docs" -- -b linkcheck

# Update parameter documentation (CI checks this is up-to-date)
# Must be run from inside docs/ (uses relative input path)
cd docs && python parse_tables.py

# Update module grid on docs homepage (CI checks this is up-to-date)
python docs/generate_module_grid.py

# Pre-commit hooks
pre-commit install
pre-commit run --all-files
```

## Code Style

- **Black** formatter: line-length 120, target Python 3.11
- **isort**: profile "black"
- **flake8**: max-line-length 120. CI hard-fails only on E9,F63,F7,F82, but also runs two non-blocking flake8 passes: a full `--exit-zero` pass and a bugbear/comprehensions pass (`--select=B,C4`). The `[dev]` extra includes `flake8-bugbear` and `flake8-comprehensions`.
- **numpydoc** validation on all code except `test/`, `exceptions.py`, `noxfile.py`, `jupyter_notebooks/`, `docs/parse_tables.py`
- Python >= 3.11 required; CI tests on 3.11, 3.12, 3.13

## Architecture

### Package Layout (important)

Scoring and datapoint code are **top-level packages**, not nested under `modules/`:

- `proteobench/score/`: `score_base.py` (`ScoreBase` ABC), `quantscoresHYE.py` (`QuantScoresHYE`), `quantscoresPYE.py` (`QuantScoresPYE`), `denovoscores.py` (`DenovoScores`)
- `proteobench/datapoint/`: `datapoint_base.py` (`DatapointBase` ABC), `quant_datapoint.py` (`QuantDatapointHYE`, `QuantDatapointPYE`), `denovo_datapoint.py` (`DenovoDatapoint`)
- `proteobench/plotting/`: `plot_generator_base.py` (`PlotGeneratorBase` ABC) plus HYE/PYE/DeNovo generators
- `proteobench/modules/`: thin module classes (quant, denovo, rescoring) that orchestrate the pipeline

So, for example, `from proteobench.score.quantscoresHYE import QuantScoresHYE` and `from proteobench.datapoint.quant_datapoint import QuantDatapointHYE`.

### Core Benchmarking Pipeline

The reference entry point is `run_benchmarking()` in `proteobench/modules/quant/benchmarking.py`. **Only two modules actually delegate to it:** `DDAQuantIonModuleQExactive.benchmarking()` calls `run_benchmarking()`, and `DDAQuantIonAstralModule.benchmarking()` calls `run_benchmarking_with_timing()` (a profiling variant returning an extra `timings` dict with per-step durations) and slices `[:3]`. The Astral DDA module also exposes a `benchmarking_2()` that returns the full 4-tuple including `timings`. **All other quant modules implement the same load/parse/score/datapoint pipeline inline within their own `benchmarking()` methods** rather than calling the shared helper.

In `run_benchmarking()`, each step is a helper wrapped with `handle_benchmarking_error(<ExceptionType>, ...)`, which maps raised exceptions to the custom exception hierarchy (see `proteobench/exceptions.py`). Note that `run_benchmarking_with_timing()` calls the underlying functions directly and does **not** apply this error wrapping.

**Pipeline steps (decorated helpers in `run_benchmarking`):**

1. **Load input** (`_load_input`, `ParseError`) - `io/parsing/parse_ion.py:load_input_file()` dispatches to format-specific loaders via the `_LOAD_FUNCTIONS` dict (17 entries). AlphaDIA has special handling: auto-detects matrix vs long format between two files and supports v1 (TSV) and v2 (parquet).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Proteobench/ProteoBench](https://github.com/Proteobench/ProteoBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ModelSkill is a Python package for evaluating model skill by comparing simulation results with observations. It's primarily used for MIKE models but supports other models as well. The package handles various types of spatial and temporal data (point observations, tracks, gridded fields, DFSU files) and provides comprehensive statistical analysis and visualization.

## Development Commands

### Package Management
This project uses `uv` for dependency management. Install dependencies with:
```bash
uv sync --group dev     # Install with dev dependencies
uv sync --group test    # Install with test dependencies
```

### Testing
```bash
just test               # Run all tests (ignores notebooks)
pytest                  # Direct pytest invocation
pytest tests/test_comparer.py::test_name  # Run specific test
pytest --disable-warnings  # Run without warnings (default in justfile)
```

### Code Quality
```bash
just check              # Run all checks: lint, typecheck, test, doctest
just lint               # Lint with ruff
just format             # Format with ruff
just typecheck          # Type check with mypy
just doctest            # Run doctests in metrics.py
just coverage           # Generate HTML coverage report
```

### Building
```bash
just build              # Run typecheck and test, then build package with uv build
uv build                # Build wheel and sdist
```

### Documentation
```bash
just docs               # Build documentation with quartodoc and quarto
                        # Located in docs/_site after building
```

### Roadmap
After editing any file in `roadmap/features/`, regenerate the overview:
```bash
uv run roadmap/scripts/generate_overview.py
```
This updates `roadmap/README.md` from the feature frontmatter.

## Coding Conventions

### Docstrings
- All docstrings use **NumPy format** (not Google or reStructuredText style)
- Include sections: Parameters, Returns, Raises, Examples, See Also, Notes as appropriate
- Example:
  ```python
  def function_name(param1, param2):
      """Short description.

      Longer description if needed.

      Parameters
      ----------
      param1 : type
          Description of param1
      param2 : type
          Description of param2

      Returns
      -------
      type
          Description of return value
      """
  ```

## Architecture

### Architecture Decision Records

Key architectural decisions are documented in the `/adr` directory. These ADRs explain the rationale behind major design choices including:
- Why mikeio is a core dependency
- The centralized metrics module approach
- The Comparer/ComparerCollection pattern
- Using xarray for internal data structures
- The model result type hierarchy
- Supporting dual plotting backends
- The four-step workflow pattern
- The options and styling system
- Factory pattern for type detection

See [/adr/README.md](adr/README.md) for the complete index.

### Core Workflow (4-Step Pattern)
The package follows a consistent 4-step workflow that users should follow:

1. **Define ModelResults** - Load/create model output data
2. **Define Observations** - Load/create observation data
3. **Match** - Spatially and temporally match observations with model results
4. **Compare** - Analyze and visualize using Comparer/ComparerCollection objects

### Key Components

#### Model Results (`src/modelskill/model/`)
Model results represent simulation output. Types inherit from a base class:
- `PointModelResult` - Fixed point timeseries (dfs0, nc, DataFrame)
- `TrackModelResult` - Moving point timeseries (dfs0, nc, DataFrame)
- `GridModelResult` - Regular gridded fields (dfs2, nc, xarray Dataset) - extractable via spatial interpolation
- `DfsuModelResult` - Unstructured mesh fields (dfsu files) - extractable via spatial interpolation
- `DummyModelResult` - Synthetic baseline for skill comparison (e.g., mean, climatology)

Use `model_result()` factory function to auto-detect type from input data.

#### Observations (`src/modelskill/obs.py`)
Observations represent measured data:
- `PointObservation` - Fixed location timeseries
- `TrackObservation` - Moving location timeseries (e.g., satellite altimetry)

Use `observation()` factory function to auto-detect type from input data.

#### Matching (`src/modelskill/matching.py`)
The `match()` function aligns observations with model results in space and time:
- Spatial matching: extracts model data at observation locations (for Grid/Dfsu)
- Temporal matching: aligns timestamps within tolerance
- Returns `Comparer` (single obs) or `ComparerCollection` (multiple obs)

Can also use `from_matched()` when data is pre-aligned.

#### Comparison (`src/modelskill/comparison/`)
The core analysis objects after matching:
- `Comparer` - Single observation vs model result(s) comparison
  - Contains matched xarray Dataset with observation and model data
  - Has `.plot` attribute (ComparerPlotter) for visualization
  - Provides `.skill()` method returning SkillTable
  - Supports filtering, selecting, and aggregation
- `ComparerCollection` - Multiple Comparers for multi-observation analysis

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DHI/modelskill](https://github.com/DHI/modelskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

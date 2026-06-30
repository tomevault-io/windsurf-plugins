---
trigger: always_on
description: `brutus` is a Python package for Bayesian inference of stellar properties, distances, and extinctions from photometry. It supports individual star fitting, cluster analysis, and 3D dust mapping.
---

# CLAUDE.md - Repository Guide for brutus

## Overview

`brutus` is a Python package for Bayesian inference of stellar properties, distances, and extinctions from photometry. It supports individual star fitting, cluster analysis, and 3D dust mapping.

**Package name on PyPI**: `astro-brutus`
**Version**: 1.0.0
**Paper**: Speagle et al. (2025), [arXiv:2503.02227](https://arxiv.org/abs/2503.02227)

## Repository Structure

```
brutus/
├── src/brutus/           # Main package source
│   ├── core/             # Stellar modeling (EEPTracks, StarGrid, Isochrone, StellarPop)
│   ├── analysis/         # Fitting (BruteForce, population modeling, photometric offsets)
│   ├── data/             # Data downloading/loading (Pooch-based, Harvard Dataverse)
│   ├── dust/             # 3D dust map utilities
│   ├── priors/           # Bayesian priors (IMF, Galactic structure, extinction)
│   ├── plotting/         # Visualization utilities
│   └── utils/            # Math, photometry, sampling utilities
├── tests/                # Test suite (pytest)
├── tutorials/            # 12 Jupyter tutorial notebooks (00-11)
├── docs/source/          # Sphinx documentation
├── pyproject.toml        # Package configuration
└── CHANGELOG.md          # Version history
```

## Development Commands

```bash
# Install in development mode
pip install -e ".[dev]"

# Run tests
pytest

# Run tests with coverage
# NUMBA_DISABLE_JIT=1 is REQUIRED: @jit functions show 0% coverage otherwise
NUMBA_DISABLE_JIT=1 pytest --cov=brutus

# Build documentation
cd docs && make html
```

Pre-commit hooks run `black`, `isort`, and `flake8`. If a commit fails on formatting, re-stage the auto-formatted files and commit again.

## Testing

Tests are in `tests/`, organized by module (`test_core/`, `test_analysis/`, etc.).

```bash
pytest tests/test_core/
pytest tests/test_analysis/test_individual.py -v
```

**Important**: When making code changes, flag any tests that may need updating. Check for:
- Tests that directly test modified functions
- Tests that use modified functions as dependencies
- Integration tests that exercise modified code paths

## Documentation

Documentation lives in `docs/source/`:
- **Getting Started**: `installation.rst`, `quickstart.rst`, `glossary.rst`
- **Scientific Background**: `scientific_background.rst`, `stellar_models.rst`, `priors.rst`, `grid_generation.rst`, `population_modeling.rst`, `photometric_offsets.rst`
- **User Guide**: `understanding_results.rst`, `faq.rst`
- **API Reference**: `api/` directory with per-module documentation
- **Development**: `changelog.rst`, `contributing.rst`

Build with `cd docs && make html`. Output in `docs/build/html/`.

## Tutorials

12 Jupyter notebooks in `tutorials/` covering individual stars, populations, grids, priors, fitting, cluster analysis, dust mapping, and photometric calibration. These use real data files (FITS/HDF5) and require downloaded model data.

## Code Style

- `black` for formatting (line length 88)
- `isort` for import sorting
- NumPy-style docstrings

## Unit Conventions and Normalization

Understanding brutus's internal conventions is essential for correct usage:

- **Flux**: "maggies" = `10^(-0.4 * m)` where `m` is the magnitude in native survey units. No absolute zeropoint offset — the conversion is purely relative.
- **Model magnitudes**: Defined at a reference distance of **1 kpc**. Apparent magnitude at distance `d` (kpc) is `m_apparent = m_model + 5 * log10(d_kpc)`.
- **Scale factor**: `scale = (d_ref / d)^2 = 1 / d_kpc^2`. Used internally to shift model flux to the observed distance.
- **Parallax**: Expected in **milliarcseconds** (mas) throughout the fitting pipeline. Survey data in arcseconds must be multiplied by 1000.
- **Extinction**: A(V) in magnitudes. The default R(V) prior is Gaussian with mean 3.32 and std 0.18.
- **Reddening vectors**: In `_get_sed_mle`, `drvecs = (1/A_V) * ∂f/∂R_V` (the A(V) factor is divided out). The Fisher information computation multiplies by A(V) to recover the true derivative `∂f/∂R_V` for correct R(V) uncertainty estimation.
- **Photometric offsets**: Multiplicative flux corrections loaded via `load_offsets()`. Applied as `flux *= offset, err *= offset` in `_setup()`. The offset uncertainties (~0.02 mag optical, ~0.03 mag NIR per Speagle et al. 2025 Table 5) are NOT automatically added to measurement errors. Users should add systematic errors in quadrature before fitting:
  ```python
  sys_err_mag = np.array([0.02, 0.02, ...])  # per filter
  sys_err_flux = flux * sys_err_mag * np.log(10) / 2.5
  err = np.sqrt(err**2 + sys_err_flux**2)
  ```

## Architecture: Grid-Based Bayesian Inference

brutus uses **systematic grid evaluation** rather than MCMC:
- Evaluates likelihood at every grid point — no convergence diagnostics needed
- For each grid point, optimizes (scale, A_V, R_V) via Gauss-Newton and computes a 3x3 Fisher information matrix
- MC samples from the Fisher-based Gaussian approximate the local posterior
- Galactic structure priors, parallax constraints, and IMF are applied as weights
- Final posterior is a weighted mixture over all grid points


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshspeagle/brutus](https://github.com/joshspeagle/brutus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

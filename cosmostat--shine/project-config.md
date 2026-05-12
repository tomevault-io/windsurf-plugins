---
trigger: always_on
description: SHINE (SHear INference Environment) is a JAX-powered framework for probabilistic shear estimation in weak gravitational lensing. It treats shear measurement as a Bayesian inverse problem: generating forward models of the sky, convolving with instrument response, and comparing to observed data to infer posterior distributions of shear parameters.
---

# CLAUDE.md - AI Assistant Guide for SHINE

## Project Overview

SHINE (SHear INference Environment) is a JAX-powered framework for probabilistic shear estimation in weak gravitational lensing. It treats shear measurement as a Bayesian inverse problem: generating forward models of the sky, convolving with instrument response, and comparing to observed data to infer posterior distributions of shear parameters.

**Status:** Early development / Alpha. Core modules (`shine.config`, `shine.inference`) and the first instrument backend (`shine.euclid`) are implemented. See `DESIGN.md` for the full architecture.

**Organization:** CosmoStat Lab (CEA / CNRS)
**License:** MIT

## Repository Structure

```
SHINE/
├── .github/workflows/       # CI/CD (Claude PR assistant + code review)
├── assets/
│   └── logo.png             # Project logo
├── configs/
│   └── euclid_vis.yaml      # Example Euclid VIS config
├── data/
│   └── EUC_VIS_SWL/         # Euclid VIS test data (Git LFS)
├── external/
│   └── GalSim/              # External GalSim dependency (placeholder)
├── notebooks/
│   └── euclid_vis_map.ipynb  # MAP fitting demo notebook
├── scripts/
│   └── test_map.py          # Standalone MAP test script
├── shine/                   # Main Python package
│   ├── __init__.py
│   ├── config.py            # Base inference configuration
│   ├── prior_utils.py       # Shared prior-parsing (config → NumPyro sample sites)
│   ├── inference.py         # Inference engine (MAP, NUTS)
│   └── euclid/              # Euclid VIS instrument backend
│       ├── config.py        # Euclid-specific configuration
│       ├── data_loader.py   # FITS data loading & source selection
│       ├── scene.py         # Multi-exposure scene model (NumPyro)
│       └── plots.py         # Diagnostic visualizations
├── tests/
│   └── test_euclid/         # Euclid module tests (15 tests)
├── CLAUDE.md                # This file
├── DESIGN.md                # Architecture & design document
├── LICENSE                  # MIT License
├── README.md                # Project overview and quick start
└── pyproject.toml           # Build configuration
```

## Key Technologies

- **JAX** — Core computation: JIT compilation, vmap vectorization, grad for HMC
- **NumPyro** — Probabilistic programming: hierarchical models, MCMC (NUTS/HMC)
- **JAX-GalSim** — Differentiable galaxy profile rendering and PSF convolution
- **BlackJAX** — Optional lower-level inference library for custom samplers

## Module Architecture

| Module | Status | Purpose |
|--------|--------|---------|
| `shine.config` | Implemented | Configuration schema (galaxy model, inference, distributions with `center: catalog`) |
| `shine.prior_utils` | Implemented | Shared prior-parsing: converts `DistributionConfig` → NumPyro sample sites |
| `shine.inference` | Implemented | Inference engine (MAP optimization, NUTS via NumPyro) |
| `shine.euclid` | Implemented | Euclid VIS instrument backend: data loading, scene model, diagnostics |
| `shine.scene_modelling` | Planned | Generic NumPyro generative model definitions |
| `shine.simulations` | Planned | Additional survey interfaces (LSST, MeerKAT) |
| `shine.morphology` | Planned | Non-parametric galaxy profiles (VAE/Diffusion) |
| `shine.wms` | Planned | Workflow management for HPC/SLURM clusters |

### `shine.euclid` — Euclid VIS Backend

The first instrument backend, providing end-to-end shear inference on Euclid VIS quadrant-level data:

- **`config.py`** — Pydantic configuration: data paths, source selection (SNR, `det_quality_flag`, size filtering), galaxy model specification via shared `GalaxyConfig` (supports `center: catalog` priors), multi-tier stamp sizes
- **`data_loader.py`** — Reads quadrant FITS files (SCI/RMS/FLG), PSF grids with bilinear interpolation, background maps, MER catalogs; computes per-source WCS positions, Jacobians, PSF stamps, and visibility
- **`scene.py`** — NumPyro generative model: renders Sersic galaxies convolved with spatially-varying PSFs via JAX-GalSim; multi-tier stamp sizes (64/128/256 px) with separate `vmap` per tier; standalone `render_model_images()` for post-inference visualization
- **`plots.py`** — 3-panel diagnostic figures (observed | model | chi residual) with configurable masking

## Build System

- **Build backend:** setuptools (>=61) with setuptools-scm (>=6.2)
- **Version:** Dynamic, managed by setuptools-scm (writes to `shine/_version.py`)
- **Python:** >=3.9 (supports 3.9 through 3.13)
- **Install:** `pip install -e .` for development

## Code Standards (from DESIGN.md Section 4.1)

When implementing code for this project, follow these conventions:

- **Formatter:** Black
- **Import sorting:** isort
- **Type hints:** Full PEP 484 compliance required
- **Docstrings:** Google-style
- **Testing:** pytest + chex (JAX-specific array shape/type testing)
- **Documentation:** Sphinx + ReadTheDocs (not yet configured)

## JAX-Specific Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CosmoStat/SHINE](https://github.com/CosmoStat/SHINE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

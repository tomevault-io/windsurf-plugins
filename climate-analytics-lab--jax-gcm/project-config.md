---
trigger: always_on
description: This is a complex codebase with many interdependencies and intricate scientific formulations. Don't assume. Don't hide confusion. Surface tradeoffs.
---

# CLAUDE.md

## Think Before Coding
This is a complex codebase with many interdependencies and intricate scientific formulations. Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:

 - State your assumptions explicitly. If uncertain, ask.
 - If multiple interpretations exist, present them - don't pick silently.
 - If a simpler approach exists, say so. Push back when warranted.
 - If something is unclear, stop. Name what's confusing. Ask.

Always document these decisions in the comments, and if appropriate in the documentation (and possibly in the high-level design documentation)

Comments should always reference the current state of the code, and explain *why* it is doing what it is doing, not how it is different to some previous version of the code (Which can get out of date and confusing)

## Project Overview

JAX-GCM (`jcm`) is a fully differentiable General Circulation Model (GCM) for atmospheric simulation, written entirely in JAX. It combines the Dinosaur spectral dynamical core with JAX implementations of ICON /ECHAM and SPEEDY atmospheric physics parameterizations. The model supports gradient-based optimization, data assimilation, and hybrid physics-ML workflows.

- **Package name:** `jcm`
- **Python:** >= 3.11 (strict requirement)
- **License:** Apache 2.0
- **Status:** Alpha (v1.0.0)

Note, the latest development work should target the `dev` branch. Clean, working releases are periodically merged into `main` and tagged. 

## Repository Structure

```
jcm/                          # Main package
├── model.py                  # Core Model class - main entry point
├── main.py                   # CLI entry point (Hydra config)
├── constants.py              # Global physical constants
├── utils.py                  # Utilities, lookup tables, and coordinate creation
├── terrain.py                # Terrain boundary conditions (orography, land-sea mask)
├── forcing.py                # Forcing boundary conditions and I/O
├── date.py                   # Date handling
├── physics_interface.py      # Physics-dynamics coupling
├── diffusion.py              # Diffusion filter
├── config/                   # Hydra configuration files
├── physics/
│   ├── physics_term.py          # PhysicsTerm base class
│   ├── composable_physics.py    # ComposablePhysics container
│   ├── speedy/                  # SPEEDY infrastructure (params, coords)
│   │   ├── speedy_terms.py      # Composable terms + speedy_physics() factory
│   │   ├── speedy_coords.py
│   │   ├── params.py
│   │   ├── physics_data.py
│   │   └── physical_constants.py
│   ├── icon/                    # ICON infrastructure (params, coords)
│   │   ├── icon_terms.py        # Composable terms + icon_physics() factory
│   │   ├── icon_physics.py      # Standalone apply_* term functions used by icon_terms
│   │   ├── icon_coords.py, icon_levels.py, icon_physics_data.py, parameters.py
│   │   ├── unit_conversions.py, forcing.py
│   │   └── constants/           # ICON physical constants
│   ├── radiation/
│   │   ├── grey_two_stream/     # ICON-style grey two-stream package
│   │   ├── rrtmgp.py
│   │   ├── nn_emulator.py + nn_emulator_scheme.py
│   │   ├── radiation_types.py, cloud_optics.py, constants.py   # shared
│   │   └── speedy_shortwave.py, speedy_longwave.py
│   ├── convection/
│   │   ├── tiedtke_nordeng/     # Tiedtke-Nordeng mass flux scheme
│   │   └── speedy_convection.py
│   ├── clouds/
│   │   ├── sundqvist.py         # Sundqvist diagnostic cloud fraction
│   │   ├── echam_1m.py          # ECHAM 1-moment microphysics
│   │   ├── speedy_humidity.py, speedy_condensation.py
│   ├── vertical_diffusion/
│   │   ├── tte_tke/             # TTE-TKE closure
│   │   └── speedy_vdiff.py
│   ├── gravity_waves/hines/     # Hines (1997) gravity wave drag
│   ├── aerosol/macv2_sp.py      # Stevens et al. (2017) MACv2-SP simple plumes
│   ├── chemistry/simple_chemistry.py
│   ├── diagnostics/wmo_tropopause.py
│   ├── surface/                 # Speedy bulk + ICON multi-tile (in surface/icon/)
│   ├── forcing/speedy_forcing.py
│   ├── orographic_correction/speedy_orographic.py
│   └── held_suarez/             # Simplified Held-Suarez forcing
│       ├── held_suarez_physics.py
│       └── utils.py             # Coordinate helpers for Held-Suarez
├── data/
│   ├── bc/                   # Boundary condition data (T30 climatology)
│   └── test/                 # Test reference data
└── *_test.py                 # Co-located unit tests
docs/                         # Sphinx documentation (RST + Furo theme)
notebooks/                    # Example Jupyter notebooks
```

## Build & Install

```bash
pip install -e .
```

Dependencies are in `requirements.txt`: dinosaur, flax, jax-datetime, tree-math, hydra-core, xarray.

## Running Tests

```bash
# Default — run in parallel across ~12 workers (pytest-xdist).
# Cuts a full sweep from ~15 min to a couple of minutes locally.
JAX_PLATFORMS=cpu pytest -n 12

# Single-process if you need ordered output or are debugging a flake
pytest

# Fast tests only (skip slow integration tests >1 min)
JAX_PLATFORMS=cpu pytest -n 12 -m "not slow"

# Specific test file
pytest jcm/model_test.py


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [climate-analytics-lab/jax-gcm](https://github.com/climate-analytics-lab/jax-gcm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

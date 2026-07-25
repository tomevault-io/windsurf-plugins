---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RadVel is an open-source Python package for modeling Keplerian orbits in radial velocity (RV) time series to detect and characterize exoplanets. It performs MAP fitting and MCMC posterior sampling with real-time convergence tests. Reference paper: [Fulton et al. (2018, PASP 130, 044504)](http://adsabs.harvard.edu/abs/2018PASP..130d4504F).

## Build & Development Commands

```bash
# Development install (build C extension + install in editable mode)
python setup.py build_ext -i && pip install -e .

# Run full test suite
pytest radvel -v --tb=short

# Run a single test function
pytest radvel/tests/test_api.py::test_k2_24 -v

# Run tests with coverage
pytest radvel --cov=radvel --cov-report=term-missing -v

# Build C Kepler solver extension only
python setup.py build_ext -i
```

The C extension (`radvel/_kepler`) is critical for performance. If it fails to build, RadVel falls back to a pure NumPy Kepler solver. Verify it loaded with: `python -c "import radvel._kepler; print('OK')"`.

On macOS, install `pytables` and `h5py` via conda before pip installing to avoid compilation issues.

## Architecture

### Core Pipeline (CLI: `radvel` command)

The CLI workflow is sequential: **fit → mcmc/ns → derive → ic_compare → tables → plot → report**

- [cli.py](radvel/cli.py) — Entry point, argument parsing, dispatches to driver functions
- [driver.py](radvel/driver.py) — High-level orchestration; each CLI subcommand maps to a driver function
- State is persisted between steps via pickle files (`_post_obj.pkl`) and a `.stat` status file

### Data Model

- [model.py](radvel/model.py) — `Parameter` (value/vary/mcmcscale/linear), `Parameters` (OrderedDict of Parameter objects), `RVModel` (forward model computing RV from parameters), `Vector` (dict↔array conversion)
- [basis.py](radvel/basis.py) — `Basis` class converts between orbital parameterizations. The **synthesis basis** (`per tp e w k`) is the internal representation used by the Kepler solver. The standard **fitting basis** is `per tc secosw sesinw k` which imposes flat priors on all orbital elements, avoids biasing K>0, and speeds MCMC convergence. Priors are assumed uniform in the fitting basis, so basis choice imposes implicit priors on orbital elements. New bases are added by updating `BASIS_NAMES` and the `to_synth`/`from_synth` methods. See `BASIS_NAMES` for the 10 supported bases.

### Kepler Solver

- [kepler.py](radvel/kepler.py) — `rv_drive()` computes RV signal from orbital elements `[per, tp, e, w, K]`. Solves Kepler's equation (M = E - e sin E) using the iterative method of Danby (1988) with third-order corrections. Uses C extension (`src/_kepler.pyx`) when available, else pure NumPy. Has a fast path for circular orbits (e=0). Eccentricity is clamped to [0, 0.99].
- The RV equation is `v_r = K[cos(nu + w) + e*cos(w)]` where the z-axis points *away* from the observer (positive RV = redshift). This is the standard observational convention (see paper Section 2.1).

### Statistical Framework

- [likelihood.py](radvel/likelihood.py) — `RVLikelihood` computes ln(L) assuming Gaussian noise: `-0.5 * sum((model - data)^2 / (err^2 + jit^2)) - sum(ln(sqrt(2*pi*(err^2 + jit^2))))`. The jitter (sigma_i) is per-instrument white noise added in quadrature. `CompositeLikelihood` sums log-likelihoods across instruments. `GPLikelihood` adds Gaussian Process correlated noise. `extra_params` holds non-Keplerian params (gamma, jitter) needed for likelihood but not the RV model itself.
- [posterior.py](radvel/posterior.py) — `Posterior` wraps Likelihood + list of Prior objects. `logprob = ln(L) + sum(ln(P_k))` for each prior P_k. If no priors defined, all are assumed uniform.
- [prior.py](radvel/prior.py) — Prior types: `Gaussian`, `Jeffreys`, `HardBounds`, `EccentricityPrior`, `PositiveKPrior`, `SecondaryEclipsePrior`. Priors are callable objects returning a single log-probability value.
- [gp.py](radvel/gp.py) — GP kernels: squared exponential, periodic, quasiperiodic, celerite-backed kernels. GP parameters prefixed `gp_` (e.g., `gp_amp`, `gp_per`, `gp_explength`, `gp_perlength`)

### Sampling

- [fitting.py](radvel/fitting.py) — MAP estimation via Powell's method in `scipy.optimize.minimize`
- [mcmc.py](radvel/mcmc.py) — MCMC via `emcee>=3` Affine Invariant sampler. Default: 8 independent ensembles of 50 walkers each, run in parallel on separate CPUs. Burn-in phase runs until G-R < 1.03, then chains restart. Production convergence requires all four criteria met for 5 consecutive checks: Gelman-Rubin < 1.01 (`maxGR`), independent samples Tz > 1000 (`minTz`), autocorrelation factor N/tau >= 40 (`minAfactor`), autocorrelation relative change < 0.03 (`maxArchange`). G-R and Tz are computed across ensembles (not within). Default MCMC step sizes: 10% of parameter value (0.001% for period).
- [nested_sampling.py](radvel/nested_sampling.py) — Nested sampling via `ultranest` (default), `dynesty`, or `PyMultiNest`

### Output

- [plot/](radvel/plot/) — `orbit_plots.py` (RV multipanel), `mcmc_plots.py` (corner, trend, autocorrelation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [California-Planet-Search/radvel](https://github.com/California-Planet-Search/radvel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

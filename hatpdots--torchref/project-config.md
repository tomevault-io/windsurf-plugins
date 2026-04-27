---
trigger: always_on
description: TorchRef is a PyTorch-based crystallographic refinement library that uses automatic differentiation and GPU acceleration for structure refinement. It follows PyTorch's `nn.Module` architecture throughout.
---

# TorchRef - Developer Guide

TorchRef is a PyTorch-based crystallographic refinement library that uses automatic differentiation and GPU acceleration for structure refinement. It follows PyTorch's `nn.Module` architecture throughout.

**Repository:** https://github.com/HatPdotS/TorchRef
**Documentation:** https://torchref.readthedocs.io/
**Version:** 0.4.1
**License:** MIT

## Quick Reference

```python
from torchref import LBFGSRefinement, ModelFT, ReflectionData, Scaler, ROOT_TORCHREF

# Load and refine
refinement = LBFGSRefinement(data_file="data.mtz", pdb="structure.pdb")
refinement.refine_xyz()         # coordinate refinement
refinement.refine_adp()         # B-factor refinement
refinement.refine(macro_cycles=5)  # full refinement
rwork, rfree = refinement.get_rfactor()
```

## Environment Setup

```bash
# Conda environment
module load anaconda && conda activate /das/work/p17/p17490/CONDA/torchref

# Python interpreter
/das/work/units/LBR-FEL/p17490/CONDA/torchref/bin/python

# Threading: auto-detected or override with
export TORCHREF_NUM_THREADS=4
```

## Building & Testing

```bash
# Install (development mode)
pip install -e ".[dev]"

# Run tests
pytest tests/                        # all tests
pytest tests/unit/                   # fast unit tests (~234 tests)
pytest tests/integration/            # integration tests (~95 tests)
pytest tests/functional/             # full workflow tests (~136 tests)
pytest -m "not gpu and not slow"     # skip GPU/slow tests

# Test with coverage
pytest tests/ --cov=torchref

# Compatibility testing (tox)
tox -e py311-default                 # Python 3.11 default deps
# See tox.ini for 27 environments (Python 3.10-3.13, various dep versions)

# Linting and formatting
black torchref/ --check              # check formatting (88 char line length)
isort torchref/ --check              # check import order
flake8 torchref/                     # lint (ignores F841, E741, E402, E722)
```

## Building Documentation

```bash
cd docs/

# Regenerate API docs from source
make apidoc

# Build HTML docs
make html
# Output: docs/_build/html/

# Run doctests in documentation
make doctest
```

Sphinx config is in `docs/conf.py`. Uses RTD theme, Napoleon (NumPy docstrings), autodoc, intersphinx (Python, NumPy, PyTorch, Pandas).

## Project Structure

```
torchref/
├── __init__.py              # Public API exports, threading config
├── config.py                # DtypeConfig singleton (dtypes.float, dtypes.int, dtypes.complex)
├── _bootstrap.py            # Threading configuration
│
├── model/                   # Atomic structure models
│   ├── model.py             # Model (base) - coordinates, B-factors, occupancies
│   ├── model_ft.py          # ModelFT - FFT structure factors, electron density
│   ├── parameter_wrappers.py  # MixedTensor, PositiveMixedTensor, OccupancyTensor
│   ├── sf_fft.py            # Structure factor calculation via FFT
│   ├── sf_ds.py             # Structure factor via direct summation
│   ├── internal_coordinates.py              # Internal coordinate systems
│   ├── segmented_internal_coordinates.py    # Segmented IC (chains)
│   ├── closed_segmented_internal_coordinates.py  # IC with chain closure
│   └── mixed_model.py       # Multi-model handling
│
├── io/                      # File I/O
│   ├── pdb.py, mtz.py, cif.py  # Format readers/writers
│   ├── data_router.py       # Auto file format detection
│   └── datasets/            # ReflectionData, FcalcData, DatasetCollection
│
├── refinement/              # Refinement framework
│   ├── base_refinement.py   # Refinement base class
│   ├── lbfgs_refinement.py  # LBFGSRefinement (primary refinement class)
│   ├── loss_state.py        # LossState - target aggregation, weights, torch.compile
│   ├── logger.py            # Progress logging
│   ├── targets/             # Target functions (xray/, geometry/, adp/, realspace, etc.)
│   ├── weighting/           # Loss weighting schemes (random, policy, component)
│   └── optimizers/          # Custom optimizers (SA, Langevin, exploratory LBFGS)
│
├── restraints/              # Geometry restraints (bonds, angles, torsions, planes, Ramachandran)
│   ├── restraints.py        # Main Restraints class
│   ├── builders.py          # Restraint construction from monomer library
│   └── library.py           # Monomer library (lazy download)
│
├── scaling/                 # Structure factor scaling
│   ├── scaler.py            # Scaler (overall scale, anisotropy, bulk solvent)
│   └── solvent.py           # Bulk solvent model
│
├── symmetry/                # Crystallographic symmetry (Cell, SpaceGroup, MapSymmetry)
├── maps/                    # Electron density maps (Map, DifferenceMap)
├── alignment/               # Patterson-based structure alignment
├── base/                    # Core math: coordinates, reciprocal, scattering, ED, FFT, kernels
├── utils/                   # Caching, device mixin, debugging, timing, serialization
├── cli/                     # 8 CLI entry points
└── data/                    # Package data (scattering tables, monomer library CIFs)
```

## Architecture

### Class Hierarchy

```
nn.Module

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HatPdotS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->

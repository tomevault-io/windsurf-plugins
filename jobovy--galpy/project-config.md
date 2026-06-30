---
trigger: always_on
description: **galpy** is a Python package for galactic dynamics that supports orbit integration in various potentials, distribution function evaluation and sampling, and calculation of action-angle coordinates. It's an astropy-affiliated package with full support for astropy's Quantity framework.
---

# Copilot Instructions for galpy

## Repository Overview

**galpy** is a Python package for galactic dynamics that supports orbit integration in various potentials, distribution function evaluation and sampling, and calculation of action-angle coordinates. It's an astropy-affiliated package with full support for astropy's Quantity framework.

- **Repository Size**: ~44MB (7.1MB galpy/, 3.3MB tests/, 18MB doc/)
- **Languages**: Python (142 files) and C (81 files for performance-critical extensions)
- **Python Versions**: Supports 3.10, 3.11, 3.12, 3.13, and 3.14
- **Key Dependencies**: numpy, scipy, matplotlib (required); GSL 1.14+ (optional, for C extensions)
- **Optional Dependencies**: astropy, astroquery, tqdm, numexpr, numba, JAX, pynbody

## Build and Installation

### Prerequisites

**ALWAYS install GSL before building on Linux/macOS** (required for C extensions):
```bash
# Ubuntu/Debian
sudo apt-get install libgsl-dev

# macOS
brew install gsl libomp
```

**Note on macOS**: Set environment variables for OpenMP support:
```bash
export CFLAGS="-I$(brew --prefix)/include -I/usr/local/opt/libomp/include"
export LDFLAGS="-L$(brew --prefix)/lib -L/usr/local/opt/libomp/lib"
```

### Installing Dependencies

**Install core dependencies BEFORE building**:
```bash
pip install --upgrade numpy scipy matplotlib setuptools cython pytest tqdm numexpr
```

Optional dependencies (install based on feature needs):
```bash
# For astropy support (Quantity with units)
pip install astropy pyerfa

# For orbit name queries
pip install astroquery

# For performance (numba)
pip install numba

# For JAX-based features
pip install jax jaxlib

# For snapshot potentials
pip install pynbody h5py pandas pytz wheel
```

### Building the Package

**CRITICAL: Use `--no-build-isolation` flag** to avoid pip timeout issues during dependency resolution:

```bash
# Standard editable install (for development)
python -m pip install --no-build-isolation -ve .

# Build in-place (for running tests without reinstalling)
python setup.py build_ext --inplace
```

**Environment Variables for Compilation**:
- `GALPY_COMPILE_NO_OPENMP=1`: Disable OpenMP support
- `GALPY_COMPILE_COVERAGE=1`: Enable gcov coverage support
- `GALPY_COMPILE_SINGLE_EXT=1`: Compile all C code into single extension (testing only)
- `GALPY_COMPILE_NO_EXT=1`: Skip C extension compilation (testing only)

**Expected Warning**: You will see a warning about "galpy action-angle-torus C library not installed" unless you manually download the Torus code from https://github.com/jobovy/Torus.git (branch: galpy) into `galpy/actionAngle/actionAngleTorus_c_ext/torus`. This is normal and does not prevent package functionality. Installing the Torus code is only necessary when making changes to `actionAngleTorus.py` or any files under `galpy/actionAngle/actionAngleTorus_c_ext/` directory.

### Verifying Installation

```bash
# Test import
python -c "import galpy; print('galpy version:', galpy.__version__)"

# Run quick test
pytest tests/test_import.py -v
```

## Testing

### Running Tests

**Full test suite takes ~50 minutes to complete**. Tests are organized by module:

```bash
# Run all tests (takes ~50 minutes)
pytest -v tests/

# Run specific test file
pytest tests/test_potential.py -v

# Run tests with specific markers
pytest tests/test_orbit.py -k 'test_energy_jacobi_conservation' -v

# Run with coverage
pytest -v tests/ --cov galpy --cov-config .coveragerc --disable-pytest-warnings
```

**Important Test Dependencies**:
- Tests in `test_orbit.py` involving `from_name` require `astroquery`
- Tests in `test_snapshotpotential.py` require `pynbody`
- Tests in `test_dynamfric.py` and `test_FDMdynamfric.py` require `numba`
- Tests in `test_sphericaldf.py` require `JAX`
- Tests in `test_quantity.py` require `astropy`

### Test Parallelization

The CI splits tests into multiple jobs for parallel execution. Reference `.github/workflows/build.yml` for the exact test file groupings if you need to run a subset of tests that matches CI behavior.

## Linting and Code Style

### Pre-commit Hooks

**Install and run pre-commit before committing**:
```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files  # Run all hooks manually
```

**Configured Hooks** (`.pre-commit-config.yaml`):
- trailing-whitespace, end-of-file-fixer
- pyupgrade (--py38-plus)
- rst-backticks
- pycln (unused imports)
- isort (import sorting, black profile)
- codespell (spell checking)
- ruff-format (code formatting)

## Project Structure

### Repository Root Files
- `setup.py`: Main build configuration (handles C extensions, GSL detection)
- `pyproject.toml`: Minimal build-system requirements, isort config
- `.pre-commit-config.yaml`: Linting/formatting configuration
- `.coveragerc`: Test coverage configuration
- `CONTRIBUTING.md`: Contributor guidelines (includes potential addition guide)
- `README.dev`: Developer notes for adding C potentials

### Main Code Structure

```
galpy/
├── __init__.py
├── actionAngle/          # Action-angle coordinate calculations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jobovy/galpy](https://github.com/jobovy/galpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

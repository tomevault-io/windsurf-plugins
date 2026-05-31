---
trigger: always_on
description: This file provides guidance to AI agents (Claude Code, GitHub Copilot, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents (Claude Code, GitHub Copilot, etc.) when working with code in this repository.

## Development Commands

### Building and Installation
```bash
# Build C++ core with CMake
cmake -B build
cmake --build build --parallel

# Install as editable Python package (rebuilds C++ automatically on changes)
pip install -e .

# Install from source
pip install git+https://github.com/proximafusion/vmecpp
```

### Testing
```bash
# Run Python tests
pytest

# Run specific test file
pytest tests/test_simsopt_compat.py

# Run C++ tests (requires separate repo)
# See: https://github.com/proximafusion/vmecpp_large_cpp_tests
```

### Code Quality
```bash
# Lint and format code
ruff check
ruff format

# Type checking
pyright

# Pre-commit checks (runs automatically on commit)
pre-commit run --all-files
```

### C++ Development (Bazel)
```bash
# Build C++ core with Bazel (from src/vmecpp/cpp/)
bazel build //...

# Run C++ tests
bazel test //vmecpp/...

# Build specific target
bazel build //vmecpp/vmec/vmec:vmec
```

### Running VMEC++
```bash
# Command line usage
python -m vmecpp examples/data/input.w7x
python -m vmecpp examples/data/w7x.json

# Run C++ standalone executable
./build/vmec_standalone examples/data/solovev.json
```

## High-Level Architecture

VMEC++ is a modern C++ reimplementation of the VMEC magnetohydrodynamic equilibrium solver with a Python interface.

### Core Components

**C++ Computational Engine** (`src/vmecpp/cpp/vmecpp/`):
- **VMEC Solver** (`vmec/vmec/`): Main iterative equilibrium solver using multigrid methods
- **Ideal MHD Model** (`vmec/ideal_mhd_model/`): Physics equations and force calculations
- **Fourier Transforms** (`common/fourier_basis_fast_*`): Fast transforms for spectral decomposition
- **Free Boundary Solver** (`free_boundary/`): NESTOR/BIEST methods for plasma-vacuum interface
- **Geometry Engine** (`vmec/fourier_geometry/`): Flux surface geometry and coordinate transformations

**Python Interface Layer** (`src/vmecpp/`):
- **VmecInput**: Pydantic model for input validation (profiles, boundary, parameters)
- **VmecOutput/VmecWOut**: Output data structures with equilibrium results
- **run()**: Primary entry point for computations
- **Free Boundary Support**: External magnetic field handling

**Python-C++ Bridge** (`src/vmecpp/cpp/vmecpp/vmec/pybind11/`):
- Automatic NumPy ↔ Eigen conversion
- Exception translation from C++ to Python
- Memory-efficient data sharing

**SIMSOPT Compatibility** (`src/vmecpp/simsopt_compat.py`):
- Drop-in replacement for SIMSOPT's Vmec class
- Optimization workflow integration
- Hot restart support for parameter scans

### Data Flow

1. **Input**: JSON (VMEC++) or INDATA (Fortran) formats → VmecInput validation → C++ VmecINDATA
2. **Computation**: Multigrid setup → Fourier decomposition → Force balance iteration → Convergence
3. **Output**: C++ results → Python data structures → Multiple formats (HDF5, NetCDF, JSON)

### Key Features

- **Zero-crash policy**: All errors reported as Python exceptions
- **Hot restart**: Initialize from previous converged state for efficient parameter scans
- **OpenMP parallelization**: Multi-threaded force calculations
- **Dual input formats**: Classic INDATA and modern JSON
- **SIMSOPT integration**: Seamless optimization workflow support

## Coding Standards and Guidelines

### C++ Code (Google Style with Physics Domain Adaptations)

**Naming Conventions**:
- **Namespaces**: `snake_case` (e.g., `vmec_algorithm_constants`)
- **Classes**: `CamelCase` (e.g., `IdealMhdModel`)
- **Functions**: `CamelCase` (e.g., `ComputeGeometry()`)
- **Constants**: `kCamelCase` (e.g., `kSignOfJacobian`)
- **Member variables**: `snake_case_` with trailing underscore
- **Physics variables**: Preserve traditional names (e.g., `bsupu_`, `iotaf_`, `presf_`)

**Function Parameters**:
- Use `m_` prefix for parameters that **will be modified** by the function
- Example: `void UpdateForces(const RadialProfiles& profiles, FourierGeometry& m_geometry)`

**Modern C++ Practices**:
- Use `std::array<>` instead of C-style arrays
- Include `<array>` header when using `std::array<>`
- Follow clang-format Google style

**Pre-commit Validation**:
- All C++ code must pass `clang-format` (Google style)
- Must pass `readability-identifier-naming` checks
- Must pass `modernize-avoid-c-arrays` checks
- Files must end with newline (`end-of-file-fixer`)

### Python Code

- **Style**: Follows `ruff` linting and formatting with line length 88
- **Type Checking**: Must pass `pyright` type validation
- **Documentation**: Use `docformatter` for consistent docstring formatting

### Development Workflow

1. **Before making C++ changes**:
   ```bash
   cd src/vmecpp/cpp
   bazel build //...  # Ensure current code builds
   ```

2. **After making changes**:
   ```bash
   # Run pre-commit checks
   pre-commit run --files path/to/modified/files

   # Build and test
   bazel build //...
   bazel test //vmecpp/...
   ```

3. **Incremental development**: Make small, focused changes that can be validated independently

## File Structure Guidelines

- **C++ core**: `src/vmecpp/cpp/vmecpp/` - Physics computations, numerical algorithms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proximafusion/vmecpp](https://github.com/proximafusion/vmecpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->

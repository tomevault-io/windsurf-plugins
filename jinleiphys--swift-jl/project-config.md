---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Julia-based nuclear physics framework implementing the Faddeev method for three-body quantum mechanical bound state calculations. The codebase specializes in solving nuclear three-body problems (like 3H tritium) using sophisticated numerical techniques including Laguerre basis functions, multi-channel coupling, and nuclear potential models.

## Development Commands

### Quick Setup (Automated)
For first-time setup or to update the environment, run the automated setup script:
```bash
./setup.sh
```
This script will:
1. Check Julia installation and install/update if needed (requires Julia >= 1.9.0)
2. Install all required Julia packages via `setup.jl`
3. Compile Fortran nuclear potential libraries in `NNpot/`

### Manual Setup (Alternative)
If you prefer manual setup or need to rebuild specific components:

**Julia Environment Setup:**
```bash
julia setup.jl
```

**Building Fortran Libraries:**
```bash
cd NNpot
make clean && make
```
This creates `libpotentials.dylib` (macOS), `libpotentials.so` (Linux), or `libpotentials.dll` (Windows).

### Build System Details
- **Fortran compiler**: `gfortran` with `-O2 -fPIC -Wall -Wextra` optimization
- **Platform detection**: Automatic selection of shared library format
- **F77/F90 compatibility**: Separate compilation flags for legacy and modern Fortran code
- **COULCC library**: Compiled automatically by `setup.sh` via `Makefile_coulcc` in `swift/`
  - Creates `libcoulcc.dylib` (macOS), `libcoulcc.so` (Linux), or `libcoulcc.dll` (Windows)
  - Provides Coulomb wavefunctions for scattering calculations

### Running Calculations
- **Main calculation**: `cd swift && julia swift_3H.jl` - Run 3H (tritium) bound state calculation
- **Script execution**: Run Julia files directly with `julia filename.jl`
- **Interactive development**: Use Jupyter notebooks in any subdirectory (*.ipynb files) for exploration
- **Memory-optimized runs**: Use `swift_3H_optimized.ipynb` for reduced memory calculations (~1-2 GB instead of 27 GB)

### Testing
- **Quick module test**: `julia NNpot/test.jl` - basic nuclear potential interface validation
- **Comprehensive test**: `julia NNpot/test_comprehensive.jl` - full system validation with multiple potentials
- **Physics validation**: `julia NNpot/test_channel_physics.jl` - channel coupling and quantum number consistency
- **Specific debugging**: Various `debug_*.jl` and `simple_*test*.jl` files for targeted testing

### Development Workflow
1. **Initial setup**: Run `./setup.sh` for automated environment setup (Julia installation, packages, Fortran libraries)
2. **Development**: Modify code, run calculations with `cd swift && julia swift_3H.jl`
3. **Interactive exploration**: Use Jupyter notebooks for debugging and method comparison
4. **Testing**: Run specific tests to validate changes (e.g., `julia NNpot/test_comprehensive.jl`)

## Core Architecture

### Module Structure
The codebase is organized into three main module directories:

1. **NNpot/**: Nuclear potential interface
   - Fortran libraries (AV18, AV14, Nijmegen) with Julia wrappers
   - `nuclear_potentials.jl`: Interface to compiled Fortran potentials
   - Dynamic library compilation via makefiles

2. **general_modules/**: Foundation components
   - `channels.jl`: Three-body channel coupling calculations with angular momentum algebra
   - `mesh.jl`: Laguerre-based numerical mesh generation for hyperspherical coordinates

3. **swift/**: Core Faddeev implementation
   - `matrices.jl`: Matrix elements for kinetic energy (T), potential (V), and coordinate transformations (Rxy)
   - `matrices_optimized.jl`: Optimized matrix computations with caching and complex scaling support
   - `scattering.jl`: Inhomogeneous scattering equation solver for three-body scattering calculations
   - `threebodybound.jl`: Direct eigenvalue solver for bound state energies
   - `MalflietTjon.jl`: Iterative Malfiet-Tjon eigenvalue solver with secant method convergence
   - `twobody.jl`: Two-body reference calculations (deuteron)
   - `laguerre.jl`: Basis function implementations
   - `Gcoefficient.jl`: Angular momentum coupling coefficients
   - `coulcc.jl`: Julia wrapper for COULCC Fortran library (Coulomb wavefunctions)

4. **3Npot/**: Three-body nuclear force models
   - `UIX.jl`: Urbana IX three-body force implementation with Y(r) and T(r) functions

### Key Physics Concepts
- **Faddeev equations**: Three-body quantum mechanics using coordinate transformations
- **Hyperspherical coordinates**: (x,y) representing relative distances in three-body system
- **Channel coupling**: Multi-channel approach with different angular momentum states
- **Nuclear potentials**: Realistic NN interactions (AV18, AV14, Nijmegen, Malfliet-Tjon)
- **Three-body forces**: Urbana IX (UIX) model with Y(r) and T(r) radial functions

### Computational Workflow

**Bound State Calculations:**
1. **Channel generation**: `α3b()` creates allowed quantum states based on conservation laws
2. **Mesh initialization**: `initialmesh()` sets up hyperspherical coordinate grids

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jinleiphys) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ACEpotentials.jl is a Julia package for creating and using atomic cluster expansion (ACE) interatomic potentials. It provides tools for fitting machine learning potentials to quantum mechanical data and using them for atomistic simulations. The package integrates with the AtomsBase ecosystem and supports linear and nonlinear ACE models.

**Current Version**: 0.9.1
**Julia Compatibility**: 1.10, 1.11 (1.11 strongly recommended)
**Active Branch**: Version 0.8+ represents a complete rewrite with new architecture

## Essential Commands

### Development Setup

The package requires the ACEregistry to be added before use:

```bash
# Activate the project
julia --project=.

# In Julia REPL, add the ACE registry
] registry add https://github.com/ACEsuit/ACEregistry

# Install dependencies
] instantiate

# Resolve and precompile
] resolve
] precompile
```

### Testing

```bash
# Run full test suite (takes ~2 hours)
julia --project=. -e 'using Pkg; Pkg.test()'

# Run main test file
julia --project=. test/runtests.jl

# Run specific test file
julia --project=. test/test_silicon.jl
julia --project=. test/models/test_ace.jl
julia --project=. test/test_migration.jl

# Run individual testset from within Julia
using ACEpotentials, Test
include("test/test_silicon.jl")
```

### Documentation

```bash
# Build documentation locally
julia --project=docs docs/make.jl

# Or from root directory
julia --project=. -e 'include("docs/make.jl")'
```

## Architecture

### Module Structure

**Top-level** (`src/ACEpotentials.jl`):
- Re-exports ACEfit for fitting functionality
- Coordinates model construction, data handling, and fitting workflows

**Core Subsystems**:

1. **Models** (`src/models/`): ACE model definitions and evaluation
   - `ace.jl`: Main ACEModel structure and constructor
   - `calculators.jl`: AtomsCalculators interface for energy/force/virial evaluation
   - `Rnl_*.jl`: Radial basis functions (learnable, splines, basis)
   - `fasteval.jl`: Optimized evaluation paths
   - `committee.jl`: Model committees for uncertainty quantification
   - `smoothness_priors.jl`: Regularization priors (algebraic, exponential, gaussian)

2. **Data Handling** (`src/atoms_data.jl`):
   - Converts AtomsBase systems to ACEfit-compatible AtomsData format
   - Handles energy/force/virial extraction with fuzzy key matching
   - Manages per-configuration weights

3. **Fitting** (`src/fit_model.jl`):
   - `acefit!()`: Main fitting interface
   - `compute_errors()`: Error analysis and RMSE computation
   - Integrates with ACEfit.jl solvers (QR, BLR, LSQR)

4. **ACE1 Compatibility** (`src/ace1_compat.jl`):
   - `ace1_model()`: Creates ACE models using familiar v0.6.x API
   - Provides backward compatibility for existing workflows

5. **JSON Interface** (`src/json_interface.jl`):
   - Model serialization/deserialization
   - Script-based fitting via parameter files
   - `make_model()`, `make_solver()`: Construct objects from dictionaries

### Key Dependencies

**ACE Ecosystem**:
- `ACEfit`: Fitting algorithms and linear solvers
- `EquivariantTensors`: Coupling coefficient generation (replaces EquivariantModels)
- `Polynomials4ML`: Polynomial basis infrastructure
- `RepLieGroups`: Representation theory for SO(3) symmetry

**External**:
- `Lux`/`LuxCore`: Neural network layer abstractions
- `Zygote`: Automatic differentiation
- `AtomsBase`/`AtomsCalculators`: Atomistic simulation interfaces
- `ExtXYZ`: Dataset I/O

### Workflow Pattern

Typical fitting workflow follows this structure:

1. **Model Construction**: Use `ace1_model()` or direct ACEModel construction
2. **Data Loading**: Load datasets via ExtXYZ (usually from artifacts)
3. **Data Preparation**: Convert to AtomsData with energy/force/virial keys
4. **Fitting**: Call `acefit!()` with solver (QR, BLR, LSQR) and optional smoothness prior
5. **Evaluation**: Use `compute_errors()` to assess fit quality
6. **Serialization**: Save/load models via JSON interface

The package maintains separation between:
- **Model definition** (what basis functions, how they're combined)
- **Model parameters** (coefficients learned from data)
- **Evaluation** (computing energy/forces from atomic configurations)

## Important Notes

### EquivariantTensors Migration ✅

**Migration Complete**: All tests passing on Julia 1.11 and 1.12

**What Changed**:
- Migrated from EquivariantModels.jl (maintenance mode) to EquivariantTensors.jl v0.3
- Updated `Project.toml`, `src/models/ace.jl`, `src/models/utils.jl`
- Replaced deprecated API calls with upstream equivalents:
  - `EquivariantModels._rpi_A2B_matrix()` → `EquivariantTensors.symmetrisation_matrix()`
  - `EquivariantModels.RPE_filter_real()` → `_rpe_filter_real()` (local implementation in `src/models/utils.jl`)

**Why Migration?**
- EquivariantModels.jl frozen (legacy support only)
- EquivariantTensors.jl actively developed with better performance and GPU support
- Future-proofs the codebase for upcoming features

**Testing Status**:
- ✅ Julia 1.11: All tests pass
- ✅ Julia 1.12: All tests pass (with 2 known issues documented below)
- ✅ Documentation build: Success

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ACEsuit/ACEpotentials.jl](https://github.com/ACEsuit/ACEpotentials.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

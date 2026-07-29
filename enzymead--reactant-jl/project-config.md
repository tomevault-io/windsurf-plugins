---
trigger: always_on
description: Reactant.jl is a Julia package that compiles Julia functions into MLIR (Multi-Level Intermediate Representation) and runs advanced optimizations, including automatic differentiation with EnzymeMLIR, to create executables for CPU/GPU/TPU via XLA. It operates as a tracing system for high-performance computing.
---

# GitHub Copilot Instructions for Reactant.jl

## Project Overview

Reactant.jl is a Julia package that compiles Julia functions into MLIR (Multi-Level Intermediate Representation) and runs advanced optimizations, including automatic differentiation with EnzymeMLIR, to create executables for CPU/GPU/TPU via XLA. It operates as a tracing system for high-performance computing.

## Language and Code Style

### Julia Code
- **Language**: Julia 1.10+
- **Code Style**: Use [Blue style](https://github.com/JuliaDiff/BlueStyle) as enforced by JuliaFormatter
- **Formatter Config**: See `.JuliaFormatter.toml` with `style = "blue"` and `always_use_return = true`
- **Type Stability**: Maintain type stability for performance-critical code
- **Broadcasting**: Prefer Julia's dot notation for array operations (e.g., `sin.(x) .+ y`)
- **Naming Conventions**: Use descriptive names; types are CamelCase, functions and variables are snake_case

### C++ Code
- **Style**: LLVM style as specified in `.clang-format`
- **Location**: C++ code lives in `deps/ReactantExtra/`
- **Build System**: Bazel is used for building C++ components

## Project Structure

```
├── src/              # Main Julia source code
│   ├── Reactant.jl   # Main module file
│   ├── TracedRArray.jl, ConcreteRArray.jl  # Core array types
│   ├── Compiler.jl   # Compilation infrastructure
│   ├── Enzyme.jl     # Automatic differentiation integration
│   ├── mlir/         # MLIR bindings and utilities
│   └── xla/          # XLA integration
├── ext/              # Package extensions (conditional loading)
├── lib/              # Sub-packages
│   └── ReactantCore/ # Core functionality
├── deps/             # Build dependencies and C++ code
│   └── ReactantExtra/ # C++ API and Bazel build files
├── test/             # Test suite
│   ├── runtests.jl   # Main test runner
│   ├── integration/  # Integration tests
│   └── nn/           # Neural network tests
└── docs/             # Documentation
```

## Core Concepts

### Array Types
- **ConcreteRArray**: Underlying buffer for device data (CPU/GPU/TPU)
- **TracedRArray**: Traced version used during compilation (no access to actual values)
- Conversion: Use `Reactant.to_rarray()` to convert Julia arrays to RArrays

### Compilation
- Use `@compile` macro to compile functions
- Compiled functions capture control flow at compile time
- Only ConcreteRArray updates are captured in compiled code

### Backends
- Default backend can be set with `Reactant.set_default_backend("cpu"/"gpu"/"tpu")`
- Supports CPU, GPU (CUDA), and TPU via XLA

## Testing

### Test Structure
- **Framework**: Uses `SafeTestsets` and Julia's built-in `Test`
- **Test Groups**: Tests are organized into three groups:
  - `core`: Basic functionality, tracing, compilation, autodiff
  - `neural_networks`: NNlib, Flux, LuxLib, Lux integration
  - `integration`: CUDA, KernelAbstractions, FFT, MPI, etc.
- **Backend Testing**: Tests can run with different backends (CPU/GPU)
- **Runtime Testing**: Tests run with both "pjrt" and "ifrt" runtimes

### Running Tests
```bash
# Run all tests
julia --project=. test/runtests.jl

# Run specific test group
julia --project=. test/runtests.jl core
julia --project=. test/runtests.jl integration
julia --project=. test/runtests.jl nn

# Run with GPU backend
REACTANT_BACKEND_GROUP=gpu julia --project=. test/runtests.jl
```

### Writing Tests
- Use `@safetestset` for isolated test environments
- Follow existing patterns in test files
- Test both forward and reverse-mode automatic differentiation where applicable
- Include edge cases and type stability checks

## CI/CD

### Workflows
- **CI**: Main test suite runs on Julia 1.10, 1.11 on multiple platforms (Ubuntu, macOS, Windows, ARM, TPU)
- **Format Check**: Enforces Julia code style via JuliaFormatter
- **Format Check (C++)**: Enforces LLVM style for C++ code via clang-format
- **Format Check (Bazel)**: Enforces Bazel file formatting with buildifier
- **Documentation**: Builds with Documenter.jl and DocumenterVitepress
- **Benchmarks**: Performance tracking on push to main

### Continuous Integration
- Tests run on push to `main` and `release-*` branches
- PRs trigger CI on relevant file changes
- Concurrency controls prevent redundant builds

## Dependencies and Extensions

### Core Dependencies
- **EnzymeCore/Enzyme**: Automatic differentiation
- **LLVM.jl**: LLVM integration
- **Functors.jl**: Recursive structure traversal
- **Adapt.jl**: Array type adaptation

### Package Extensions
Reactant uses Julia's package extensions for optional integrations:
- CUDA, KernelAbstractions for GPU computing
- NNlib for neural network primitives
- Zygote for alternative AD (Julia < 1.12)
- MPI for distributed computing
- AbstractFFTs, SpecialFunctions, etc.

### Adding Dependencies
- Update `Project.toml` [deps] for required dependencies
- Add to [weakdeps] and create extension in `ext/` for optional dependencies
- Specify version bounds in [compat] section

## Development Workflow

### Setting Up Development Environment
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnzymeAD/Reactant.jl](https://github.com/EnzymeAD/Reactant.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

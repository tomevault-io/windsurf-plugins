---
trigger: always_on
description: This repository implements a modular PyTorch library for solving sparse linear systems `Ax = b`, mainly for scientific computing and PDE workloads.
---

# Repository Guide for Agents

## What this repository is

This repository implements a modular PyTorch library for solving sparse linear systems `Ax = b`, mainly for scientific computing and PDE workloads.

The package name is `pytorch_sparse_solver`, and the code uses a `src/` layout:

- `src/pytorch_sparse_solver/`: main library
- `src/run.py`: unified test and benchmark runner
- `FVM_example/`: application examples, not core package code
- `src/docs/`: installation and module architecture notes

The central idea is "multiple solver backends behind one interface":

- Module A: pure PyTorch iterative solvers
- Module B: NVIDIA AMGX via `pyamgx`
- Module C: cuDSS direct solve via `torch.sparse.spsolve`
- Unified interface: `SparseSolver` in `solver.py`

## Functional summary

### Module A: pure PyTorch iterative solvers

Primary file: `src/pytorch_sparse_solver/module_a/torch_sparse_linalg.py`

Implements:

- `cg`
- `bicgstab`
- `gmres`
- differentiable variants of all three solvers

Important characteristics:

- inspired by JAX `scipy.sparse.linalg`
- accepts dense tensors and callable linear operators
- includes PyTree utilities in `module_a/torch_tree_util.py`
- supports implicit differentiation through custom autograd logic
- the standard public `cg` / `bicgstab` / `gmres` APIs now auto-attach implicit-differentiation backward logic for plain 2D tensor systems when `A` or `b` requires gradients
- defaults to high precision (`float64` / `complex128`)
- has optional `torch.compile` plumbing, but `_JIT_ENABLED` is currently `False`

Important nuance:

- legacy `*_differentiable` entry points still exist for compatibility
- but normal users should prefer the standard solver names first

This is the most self-contained backend and the safest place to start debugging.

### Module B: AMGX backend

Primary file: `src/pytorch_sparse_solver/module_b/torch_amgx.py`

Implements GPU-accelerated solves through NVIDIA AMGX and `pyamgx`, with custom backward logic based on implicit differentiation.

Current public entry points:

- `amgx_cg`
- `amgx_bicgstab`
- `amgx_gmres`
- `amgx_amg`

Key constraints:

- requires CUDA
- requires AMGX installed separately
- requires `pyamgx`
- code converts tensors to CPU NumPy / SciPy CSR structures to interact with AMGX
- the public wrapper now returns gradients to the original input `A` and `b`, not only to internal CSR value buffers

Important nuance:

- `amgx_amg` is the direct AMGX AMG solver
- `amgx_cg` / `amgx_bicgstab` / `amgx_gmres` are Krylov-method frontends implemented through AMGX, not the same thing as `amgx_amg`

Treat this as an optional backend. Always gate usage with availability checks.

### Module C: cuDSS direct solver

Primary file: `src/pytorch_sparse_solver/module_c/cudss_solver.py`

Implements a direct sparse solve using `torch.sparse.spsolve`, plus a custom backward pass.

Key constraints:

- requires CUDA
- requires a PyTorch build with cuDSS support
- effectively expects CSR input for the direct path

This backend is only for `direct` solving in the unified interface.

### Unified interface

Primary file: `src/pytorch_sparse_solver/solver.py`

Core public objects:

- `SparseSolver`
- `solve`
- `cg`
- `bicgstab`
- `gmres`
- `amg`
- `direct_solve`
- `SolverResult`

Backend selection behavior:

- `method="direct"` requires Module C
- `method="amg"` requires Module B
- iterative methods prefer Module B on CUDA if available
- otherwise Module A is the normal fallback

## Important paths

- `README.md`: user-facing overview and installation notes
- `pyproject.toml`: packaging metadata and dependency declarations
- `src/pytorch_sparse_solver/__init__.py`: public exports
- `src/pytorch_sparse_solver/utils/availability.py`: authoritative backend availability checks
- `src/pytorch_sparse_solver/utils/matrix_utils.py`: sparse format conversion and matrix builders
- `src/pytorch_sparse_solver/tests/`: correctness tests and benchmark code
- `src/pytorch_sparse_solver/tests/test_gpu_validation.py`: authoritative GPU integration validation target
- `FVM_example/LDC_by_torchsp/ldc_solver.py`: compatibility wrapper to the Module A LDC solver
- `FVM_example/LDC_by_torchsp/ldc_solver_common.py`: shared LDC/FVM implementation
- `FVM_example/LDC_by_torchsp/ldc_solver_module_a.py`: Module A LDC variant
- `FVM_example/LDC_by_torchsp/ldc_solver_module_b.py`: Module B LDC variant
- `FVM_example/LDC_by_torchsp/ldc_solver_module_c.py`: Module C LDC variant
- `FVM_example/LDC_by_torchsp/ldc_solver_module_d.py`: unified-interface LDC variant

## How the repository is intended to be used

Typical usage patterns:

- import the unified solver and let it choose an available backend
- use Module A directly for pure PyTorch iterative solves
- use Module B or C only when the local environment has the required NVIDIA stack
- if you want true AMG behavior on NVIDIA AMGX, use `amgx_amg` or unified `method="amg"`
- use the LDC examples as application/demo coverage for all four backend styles

The project is aimed at sparse linear algebra inside numerical simulation workflows, especially Poisson-like systems and PDE discretizations.

## Test and benchmark entry points

Primary commands:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Litianyu141/Pytorch-Sparse-Linalg-torch-amgx.cg.bicg.gmres](https://github.com/Litianyu141/Pytorch-Sparse-Linalg-torch-amgx.cg.bicg.gmres) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->

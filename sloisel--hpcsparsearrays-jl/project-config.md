---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commit Messages

Do NOT add "Co-Authored-By" lines or any other self-attribution to commit messages. Do NOT advertise Claude or Anthropic in commits. Keep commit messages focused on describing the changes only.

## Build and Test Commands

```bash
# Run all tests (spawns MPI processes automatically via test harness)
julia --project=. -e 'using Pkg; Pkg.test()'

# Run a specific MPI test directly (for debugging)
mpiexec -n 4 julia --project=. test/test_matrix_multiplication.jl
mpiexec -n 4 julia --project=. test/test_transpose.jl
mpiexec -n 4 julia --project=. test/test_addition.jl
mpiexec -n 4 julia --project=. test/test_lazy_transpose.jl
mpiexec -n 4 julia --project=. test/test_vector_multiplication.jl
mpiexec -n 4 julia --project=. test/test_dense_matrix.jl
mpiexec -n 4 julia --project=. test/test_sparse_api.jl
mpiexec -n 4 julia --project=. test/test_blocks.jl
mpiexec -n 4 julia --project=. test/test_utilities.jl
mpiexec -n 4 julia --project=. test/test_local_constructors.jl
mpiexec -n 4 julia --project=. test/test_indexing.jl
mpiexec -n 4 julia --project=. test/test_factorization.jl

# GPU tests run automatically when Metal.jl is available
# Tests are parameterized over (scalar type, backend) configurations

# Precompile the package
julia --project=. -e 'using Pkg; Pkg.precompile()'
```

## MPI Configuration

By default, MPI.jl uses MPItrampoline_jll. On some Linux clusters, this causes MUMPS to hang during the solve phase. If you experience hangs with multi-rank MUMPS tests, switch to MPICH_jll:

```julia
using MPIPreferences
MPIPreferences.use_jll_binary("MPICH_jll")
```

This creates/updates `LocalPreferences.toml` (which is gitignored). Restart Julia after changing MPI preferences.

## GPU Support

GPU acceleration is supported via Metal.jl (macOS) or CUDA.jl (Linux/Windows) as package extensions.

### Type Parameters

- `HPCVector{T,AV}` where `AV` is `Vector{T}` (CPU), `MtlVector{T}` (Metal), or `CuVector{T}` (CUDA)
- `HPCMatrix{T,AM}` where `AM` is `Matrix{T}` (CPU), `MtlMatrix{T}` (Metal), or `CuMatrix{T}` (CUDA)
- `HPCSparseMatrix{T,Ti,AV}` where `AV` is `Vector{T}` (CPU), `MtlVector{T}`, or `CuVector{T}` for the `nzval` array
- Type aliases: `HPCVector_CPU{T}`, `HPCMatrix_CPU{T}`, `HPCSparseMatrix_CPU{T,Ti}` for CPU-backed types

### Creating Zero Arrays

Use `Base.zeros` with the full parametric type or type alias:

```julia
# CPU zero arrays
v = zeros(HPCVector{Float64,Vector{Float64}}, 100)
v = zeros(HPCVector_CPU{Float64}, 100)  # Equivalent using type alias

A = zeros(HPCMatrix{Float64,Matrix{Float64}}, 50, 30)
A = zeros(HPCMatrix_CPU{Float64}, 50, 30)

S = zeros(HPCSparseMatrix{Float64,Int,Vector{Float64}}, 100, 100)
S = zeros(HPCSparseMatrix_CPU{Float64,Int}, 100, 100)

# GPU zero arrays (requires Metal.jl or CUDA.jl loaded)
using Metal
v_gpu = zeros(HPCVector{Float32,MtlVector{Float32}}, 100)
A_gpu = zeros(HPCMatrix{Float32,MtlMatrix{Float32}}, 50, 30)

# Or with CUDA
using CUDA
v_gpu = zeros(HPCVector{Float64,CuVector{Float64}}, 100)
A_gpu = zeros(HPCMatrix{Float64,CuMatrix{Float64}}, 50, 30)
```

### CPU Staging

MPI communication always uses CPU buffers (no GPU-aware MPI). GPU data is staged through CPU:

1. GPU vector data copied to CPU staging buffer
2. MPI communication on CPU buffers
3. Results copied back to GPU

Plans (`VectorPlan`, `DenseMatrixVectorPlan`, `DenseTransposeVectorPlan`) include:
- `gathered::AV` - buffer matching input type
- `gathered_cpu::Vector{T}` - CPU staging buffer
- `send_bufs`, `recv_bufs` - always CPU for MPI

### Sparse Operations with GPU Vectors

Sparse matrices remain on CPU (Julia's `SparseMatrixCSC` doesn't support GPU arrays). For `A * x` where `x` is GPU:
1. Gather `x` elements via CPU staging
2. Compute sparse multiply on CPU
3. Copy result to GPU via `_create_output_like()`

### Extension Files

- `ext/HPCSparseArraysMetalExt.jl` - Metal extension with DeviceMetal backend support
- `ext/HPCSparseArraysCUDAExt.jl` - CUDA extension with DeviceCUDA backend support and cuDSS multi-GPU solver
- Loaded automatically when `using Metal` or `using CUDA` before `using HPCSparseArrays`
- Use `to_backend(obj, target_backend)` to convert between backends

### CUDA-Specific: cuDSS Multi-GPU Solver

The CUDA extension includes `CuDSSFactorizationMPI` for distributed sparse direct solves using NVIDIA's cuDSS library with NCCL inter-GPU communication:

```julia
using CUDA, MPI
MPI.Init()
using HPCSparseArrays

# Each MPI rank should use a different GPU
CUDA.device!(MPI.Comm_rank(MPI.COMM_WORLD) % length(CUDA.devices()))

# Create factorization (LDLT for symmetric, LU for general)
F = cudss_ldlt(A)  # or cudss_lu(A)
x = F \ b
finalize!(F)  # Required: clean up cuDSS resources
```

**Important cuDSS notes:**
- Requires cuDSS 0.4+ with MGMN (Multi-GPU Multi-Node) support
- NCCL communicator is bootstrapped automatically from MPI
- `finalize!(F)` must be called to avoid MPI desync during cleanup
- Known issue: tridiagonal matrices with 3+ rows per rank may fail (cuDSS bug reported to NVIDIA)

### Writing Unified CPU/GPU Functions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sloisel/HPCSparseArrays.jl](https://github.com/sloisel/HPCSparseArrays.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

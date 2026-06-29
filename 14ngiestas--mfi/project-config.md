---
trigger: always_on
description: - **NEVER edit `.f90` files directly** — they are generated artifacts from `.fpp`/`.fypp` sources.
---

# MFI — Agent Instructions

## Critical Rules

- **NEVER edit `.f90` files directly** — they are generated artifacts from `.fpp`/`.fypp` sources.
- Always modify `.fpp`/`.fypp` macros, then run:
  ```sh
  make clean && make
  ```
- `.f90` files are gitignored (line 44 of `.gitignore`) but committed to deployment branches by CI.

## Build & Test Commands

```sh
# Enter dev shell (provides gfortran, fpm, fypp, BLAS, LAPACK)
nix develop             # cpu-only
nix develop .#gpu-modern   # CUDA 12.3
nix develop .#gpu-legacy   # CUDA 11.8

# CPU-only (default)
make
fpm test

# GPU/cuBLAS
make
fpm build --profile cublas
fpm test --profile cublas
```

> **Note:** `gpu-modern` CI uses `--profile debug` to avoid a gfortran -O2 optimizer
> bug (fixed in gfortran 15.2.0). See `BUGS.md`.

## Nix Flake

A single `flake.nix` provides all dev shells (replaces old `shells/*.nix`):

| Shell | Command | CUDA |
|-------|---------|------|
| `cpu-only` | `nix develop .#cpu-only` | — |
| `gpu-modern` | `nix develop .#gpu-modern` | 12.3 |
| `gpu-legacy` | `nix develop .#gpu-legacy` | 11.8 |
| default | `nix develop` | — (same as cpu-only) |

- nixpkgs pinned to **24.11** (last version with CUDA 11.8/12.3)
- fpm 0.13.0 via inline overlay (PR #506818 in nixpkgs) — remove once merged
- gfortran, fpm, fypp, pkg-config all provided by the flake
- CI uses `magic-nix-cache-action` for fast cached builds
- Temp make files (`.mfi_*`, `.f77_*, *.tmp`) are gitignored

## Branch Model

| Branch | Purpose | Deployment Target |
|--------|---------|-------------------|
| `main` | Primary development (CPU + cuBLAS via features) | `mfi-fpm` (via CI) |
| `impl/cublas` | GPU/experimental staging | `mfi-cublas` (via CI) |
| `mfi-fpm` | CPU-only deploy artifact (`.f90` + `.toml` only) | — |
| `mfi-cublas` | GPU deploy artifact (`.f90` + `.toml` only) | — |

**CI triggers:**
- Push to `main` → full test matrix → deploy to `mfi-fpm`
- Push to `impl/cublas` → full test matrix → deploy to `mfi-cublas`
- PR to `main` → full test matrix (no deploy)
- Other branches → manual dispatch only

**To deploy:** commit changes, push to the corresponding branch. CI handles the rest.

## Code Generation Architecture

### Macro Files (edit these)
- `common.fpp` — Core fypp macros: type prefixes (`s`,`d`,`c`,`z`), `@:optional`, `@:defaults`, interface generators
- `cublas.fpp` — CUDA/cuBLAS v2 C-interop interfaces (`pure` + `VALUE` on all `bind(c)` args), `@:allocate`, `@:deallocate`, `@:set_matrix`, `@:get_matrix` macros, cuBLAS constants
- `extensions.fpp` — cuBLAS handle lifecycle (`mfi_cublas_handle_ensure`, `mfi_cublas_finalize`), execution mode control (`mfi_force_gpu`, `mfi_force_cpu`), `mfi_cublas_error`

### Source Macros (edit these)
- `src/mfi/blas/*.fypp` — MFI modern wrapper implementations
- `src/f77/blas/*.fypp` — F77 interface declarations
- `src/mfi/lapack/*.fypp` — LAPACK modern wrappers
- `src/f77/lapack/*.fypp` — LAPACK F77 interfaces

### Generated (do not edit)
- `src/f77/blas.f90`, `src/mfi/blas.f90`, `src/f77/lapack.f90`, `src/mfi/lapack.f90`
- All `test/**/*.f90` files

## Naming Conventions

| Name | Kind | Purpose |
|------|------|---------|
| `MFI_CUBLAS` | Preprocessor macro | Enables cuBLAS code at compile time (set by fpm `cublas` feature) |
| `MFI_USE_CUBLAS` | Internal variable + env var | Runtime GPU dispatch flag (read from env var on lazy init) |
| `MFI_EXTENSIONS` | Preprocessor macro | Enables BLAS extension routines (iamin, iamax, lamch) |
| `MFI_LINK_EXTERNAL` | Preprocessor macro | Links external BLAS extensions |

## Purity: Why `pure` on GPU Wrappers is Correct

**All MFI BLAS wrappers (`mfi_gemm`, `mfi_gemv`, `mfi_trsm`, `mfi_trmm`) and all `bind(c)` CUDA/cuBLAS interfaces are `pure`. This is intentional and semantically correct.**

Do NOT remove `pure` from these routines. Reasons:

1. **`error stop` is allowed in `pure` procedures** — permitted by Fortran 2008. The fact that a routine may abort on failure does not make it impure.

2. **GPU alloc → compute → dealloc is semantically pure from Fortran's perspective** — The CUDA device memory (allocated via `cudaMalloc`, freed via `cudaFree`) is opaque to the Fortran compiler. No Fortran-visible state is modified. This is exactly the same pattern as local `allocate`/`deallocate` inside a `pure` CPU function.

3. **`bind(c)` + `pure` is valid** — The compiler cannot verify purity of external C code, so it trusts the declaration. That's the whole point: you're asserting to the compiler that the side effects are not observable from Fortran.

4. **Dependent projects need this** — Projects like CheesyHam call these wrappers from `pure` contexts. Removing `pure` breaks their compilation.

The pattern:
```fortran
pure subroutine mfi_sgemm(a, b, c, ...)
    ! allocate GPU memory (opaque to Fortran)
    ! call cuBLAS (external C, compiler trusts purity claim)
    ! copy result back (no Fortran state modification)
    ! free GPU memory (opaque to Fortran)
    ! return — no observable side effects
end subroutine
```
is identical to:
```fortran
pure function foo(x) result(y)
    real, allocatable :: tmp(:)
    allocate(tmp(size(x)))    ! allowed in pure
    tmp = x * 2.0
    y = sum(tmp)
    deallocate(tmp)           ! allowed in pure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [14NGiestas/mfi](https://github.com/14NGiestas/mfi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

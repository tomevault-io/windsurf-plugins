---
trigger: always_on
description: Project-specific REQUIREMENTS for anyone (human or agent) working on PureBLAS. This is a
---

# PureBLAS.jl — agent guidelines

Project-specific REQUIREMENTS for anyone (human or agent) working on PureBLAS. This is a
**multi-session, long-horizon** project — preserve knowledge here and in `ROADMAP.md` (the
canonical status + next steps), not just in a chat transcript.

PureBLAS is the second package of the **Pure Julia Ecosystem** ("Pure"): pure-Julia replacements
for Julia's non-Julia default libraries. PureFFT.jl is the first (sibling repo). PureBLAS replaces
OpenBLAS/MKL. Mirror PureFFT's conventions (layout, ReTestItems, StrictMode, TypeContracts, trim,
DocumenterVitepress).

## What PureBLAS is (architecture)

Pure-Julia BLAS plugged into Julia **two ways**, both first-class:
1. **Native API (Mode 2)** — `PureBLAS.axpy!(y,a,x)`, `PureBLAS.dot(x,y)`, … Direct Julia calls,
   no `ccall` boundary, so they are **AD-traceable** (ForwardDiff/Enzyme/ChainRules). This is the
   higher-value mode — opaque OpenBLAS `ccall`s never allowed differentiation through BLAS.
2. **LBT drop-in (Mode 1)** — reroute LinearAlgebra's BLAS/LAPACK to PureBLAS with ONE call,
   MKL.jl-style. **`PureBLAS.activate()`** registers in-process `@cfunction` pointers to the native
   `@ccallable` kernels via `lbt_set_forward` (`cabi_forward.jl`, 123 symbols); after it, `A*B`,
   `mul!`, `cholesky`, `qr`, `svd`, `LinearAlgebra.BLAS.*` all dispatch to PureBLAS. `deactivate()`
   restores OpenBLAS. This runs in the LIVE process against its own runtime — no `.so`, no double-init.
   The SAME `@ccallable` symbols also `juliac --trim` → `libpureblas.so` for **non-Julia hosts**
   (C/C++/Rust; self-inits its embedded runtime — see `juliac/ctest.c`) and prove trim-compatibility.
   **DON'T re-chase:** `BLAS.lbt_forward(libpureblas.so)` from *inside* live Julia still aborts (signal
   6 — the juliac lib double-inits the embedded libjulia); that path is only for non-Julia hosts. In
   Julia, use `activate()` (@cfunction registration), NOT `.so` forwarding. See `ROADMAP.md` "In-process
   LBT forwarding".

Both modes share ONE set of low-level kernels. Source map:
`core.jl` (accessors `_ld`/`_st!` over Ptr AND AbstractVector, lassq, |·|) · `cpuinfo.jl`
(SIMD width, const-folded, trim-safe) · `simd_kernels.jl` (SIMD.jl fast paths) · `level1.jl`
(low-level `(n,…,inc)` kernels) · `level2.jl` (gemv/ger/symv/hemv/trmv/trsv) · `level2_packed.jl`
(spmv/hpmv/tpmv/tpsv) · `level2_banded.jl` (gbmv/sbmv/hbmv/tbmv/tbsv) · `gemm.jl` (L3) ·
`contracts.jl` (TypeContracts `AbstractBLAS1`/`AbstractBLAS2`) · `backend.jl`
(`SIMDBackend`, Mode 2) · `native.jl` (bare API) · `cabi.jl`/`cabi_l2.jl`/`cabi_l3.jl`/`cabi_lapack.jl`
(`@ccallable` ABI) · `cabi_forward.jl` (in-process LBT `@cfunction` forward registry) · `lbt.jl`
(`activate`/`deactivate`).

## Hard requirements (MUST follow)

1. **Performance gate: ≥ 1.00× `max(OpenBLAS, AOCL)`, non-negotiable — evaluated on the ratio ROUNDED
   TO TWO SIGNIFICANT DIGITS.** Per-machine (per µarch on the fleet). Beat it where possible. BLAS-1 is
   bandwidth-bound (easy parity); the real fight is M2 `dgemm`.
   The threshold is unchanged at 1.00; what is specified is the PRECISION of the comparison. `0.995`
   rounds to `1.0` and PASSES; `0.9949` rounds to `0.99` and FAILS. Rationale: per-cell machine-state
   drift on this fleet runs ~1–6% (every cached arm stores the anchor it was measured under), so the
   third digit is not adjudicable, and the rounded figure is what the published tables and plots show —
   the verdict must agree with the number a reader can see. **The criterion lives in ONE place,
   `bench/gatecrit.jl` (`gate_pass` / `GATE_MIN`); never re-spell the comparison inline.** Consumers:
   `plots.jl`, `gate_misses.jl`, `gate_gaps.jl`, `coverage_ops.jl`, `coverage_routing.jl`,
   `gate_verdict.jl` (compares in log space against `log(GATE_MIN)`), and `adjudicate.sh` (carries the
   literal, kept in sync by comment — inline julia cannot `include`).
2. **SIMD.jl for kernels** (`Vec`, `vload`/`vstore`, `muladd`). Real unit-stride dense → SIMD fast
   path; everything else (complex, strided, any other `T<:Number`) → generic scalar loop.
3. **Generic over `T<:Number`.** ONE kernel implementation covers s/d/c/z (and ForwardDiff.Dual,
   etc.). The generic scalar path is what makes Mode 2 differentiable — do not specialize it away.
4. **Trim-compatible** (juliac --trim builds the .so). No runtime `eval`/`invokelatest`, no
   `Vector{Any}` at runtime, no CpuId ccall at runtime (bake detection into consts — see cpuinfo.jl).
   Verify with TrimCheck `@validate`.
5. **TypeContracts for interfaces** (`AbstractBLAS1` in contracts.jl). Backends carry explicit
   return-type annotations so inference matches the contract; eliminated by the trimmer.
6. **`nrm2` uses LAPACK scaled accumulation (lassq)** — overflow/underflow safe. Correctness
   boundary; never simplify to `sqrt(sum(abs2))`.
7. **Adapt to the CPU via compile-time detection, NOT manual flags.** The fleet spans different
   ISAs, cache sizes, and microarchitectures. PureBLAS detects the build machine with
   **`CpuId` / `HostCPUFeatures` / `CPUSummary`** and bakes the result into **const-folded, trim-safe
   consts** (`cpuinfo.jl`: `_SIMD_BYTES`/`_vwidth`, `_L1_BYTES`, `_INTEL_AVX2`, …), each **overridable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [el-oso/PureBLAS.jl](https://github.com/el-oso/PureBLAS.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->

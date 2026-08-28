---
trigger: always_on
description: This repository is an upstream-backed fork of PDAL for a CUDA-native engine
---

# GPUPDAL agent guide

This repository is an upstream-backed fork of PDAL for a CUDA-native engine
whose public drop-in CLI is `gpupdal` and whose internal C++ namespace remains
`pdg`. Read this file first, then read
`spec.md`, `IMPLEMENTATION_PLAN.md`, `DECISIONS.md`, and
`docs/testing-strategy.md`. Read `docs/stage-coverage.md` before changing stage
selection, fallback, or public coverage claims.

## Non-negotiable priorities

1. The default compatibility mode reproduces the pinned upstream PDAL oracle
   byte-for-byte, including output files, metadata, point order, stdout,
   stderr, and exit status where applicable.
2. Speed is optimized subject to that compatibility contract. A stage that
   cannot yet be exact must use the upstream/host fallback in compatibility
   mode. Relaxed floating-point or ordering semantics belong behind an
   explicit `--fast` mode and must never silently change defaults.
3. No performance claim is accepted without a same-machine PDAL baseline,
   profile evidence, and an append-only entry in `BENCHMARKS.md`.
4. GPU stages share the planner-owned spatial index. A stage must never build
   a private k-d tree, grid, or BVH.
5. Preserve upstream licensing and attribution. Never introduce LASzip or
   other incompatible code.
6. The PDAL fork is the product. Preserve every configured PDAL command and
   stage. Report functional and GPU-native coverage separately; never call a
   host fallback GPU-native.
7. **The goal is speed, not GPU.** CUDA is one means among several. Success is
   measured wall-clock time on the reference pipelines in
   `bench/pipelines/reference/` against the pinned oracle on identical
   hardware — not by how many stages have a CUDA path. Host-side work
   (threading, algorithms, I/O, allocation) is first-class and counts as a win
   even when no GPU is involved. A CUDA path is built or kept only where it is
   measured faster end-to-end; an exact-but-slower CUDA lane stays host-selected
   and is never presented as an acceleration. Functional coverage remains
   catalog-wide and the exact-output contract is unchanged; catalog-wide *CUDA*
   coverage is no longer a release criterion. See D0208, which supersedes the
   corresponding parts of D0019.

The exact-output requirement supersedes the looser order-insensitive and
numeric-tolerance language in `spec.md`. See decision D0002.

## Upstream and branch policy

- `upstream` is `https://github.com/PDAL/PDAL.git`.
- `origin` is the hosted fork at `https://github.com/zymazza/PDAL.git`.
- The compatibility oracle is pinned in `cmake/pdg-oracle.cmake`; update it
  only with a decision entry and regenerated golden manifests.
- GPU work happens on `gpu-main` or a short-lived branch based on it.
- Keep upstream PDAL buildable. Its library and CLI are the differential
  oracle until a separately pinned oracle container replaces them.
- Do not copy an upstream implementation into CUDA without retaining BSD
  attribution in `NOTICE` and citing the relevant source or paper.

## Required work loop

1. Identify the smallest phase item from `IMPLEMENTATION_PLAN.md`.
2. Write the differential or numeric test first. For exact stages, capture the
   upstream bytes and diagnostics before implementing the accelerated path.
3. Implement the smallest complete vertical slice: descriptor, planner data,
   implementation, fallback, tests, benchmark entry, and documentation.
4. Build and run the narrow tests, then the relevant corpus tier.
5. Run sanitizers appropriate to the change. CUDA code must pass
   compute-sanitizer memcheck and racecheck on a GPU runner.
6. Profile before optimizing. Record the limiting resource and roofline gap.
7. Append results or deviations to `BENCHMARKS.md` or `DECISIONS.md`; never
   rewrite prior entries.

## Definition of done for a stage

- PDAL-compatible stage name, options, defaults, errors, and descriptor.
- Correct planner metadata: reads, writes, kind, index request, maximum radius,
  ordering behavior, and XYZ invalidation.
- Default mode is byte-identical to the pinned oracle across its fixture
  matrix, or explicitly selects the host fallback.
- `--fast` differences, if any, are documented and separately tested.
- NVTX ranges cover the stage and every CUDA runtime call uses
  `PDG_CUDA_CHECK`.
- Unit, differential, property, determinism, and malformed-input tests exist
  as applicable.
- An nvbench or google-benchmark case and same-machine upstream baseline exist.
- The public stage page documents parity, fallback behavior, and performance.
- Mixed pipelines retain unchanged PDAL stages in-process when execution mode
  and ordering are proven; otherwise delegate the original pipeline before
  data or output side effects.
- Host sanitizers and compute-sanitizer are clean.

## CUDA and C++ rules

- C++20 host code and the nvcc-supported C++20 subset only.
- CUDA 12.4 or newer. RTX 4090/SM 89 is the performance reference, never the
  compatibility floor. Release builds use every real architecture supported
  by their CUDA compiler plus PTX for its newest target. CUDA 12.x supplies the
  legacy-compatible SM 50–90 artifact; CUDA 13+ supplies the current SM 75+
  artifact, including post-Hopper targets known to that compiler. A generated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zymazza/GPUPDAL](https://github.com/zymazza/GPUPDAL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

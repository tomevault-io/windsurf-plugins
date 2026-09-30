---
trigger: always_on
description: Loopy is a Python library for transformation-based generation of high-performance array-oriented code for CPUs and GPUs. Users describe a computation and its iteration domains, then apply explicit transformations for tiling, parallelization, prefetching, layout changes, unrolling, vectorization, and instruction-level parallelism.
---

# AGENTS.md

## Project overview

Loopy is a Python library for transformation-based generation of high-performance array-oriented code for CPUs and GPUs. Users describe a computation and its iteration domains, then apply explicit transformations for tiling, parallelization, prefetching, layout changes, unrolling, vectorization, and instruction-level parallelism.

Loopy is intentionally not a general-purpose programming language. Its main application areas include linear algebra, convolutions, N-body interactions, finite-element/finite-difference solvers, and other structured array computations.

Important dependencies and representations:

- `namedisl` and `islpy`: Presburger/polyhedral sets, maps, (possibly piece-wise) quasi-affine expressions.
- `pymbolic`: scalar and array expression trees and mapper infrastructure.
- `pytools`: caching, persistent hashes, and utility infrastructure.
- `cgen`/`genpy`: generated C-family and Python syntax trees.
- `pyopencl`, optionally: OpenCL execution and runtime integration.

The package requires Python 3.10 or newer. Static-analysis configuration targets Python 3.12.

## Repository layout

### Core package

- `loopy/__init__.py`
  - Main public API and re-exports.
  - Check whether a new public class or transform must be exported here.

- `loopy/kernel/__init__.py`
  - `LoopKernel`, kernel state, core kernel-level queries, grid sizing, variable lookup.

- `loopy/kernel/data.py`
  - Kernel arguments, temporaries, address spaces, iname tags, hardware-axis tags, and related descriptors.
  - Current `AddressSpace` values are `PRIVATE`, `LOCAL`, and `GLOBAL`.

- `loopy/kernel/array.py`
  - Current array shape/layout representation.
  - `ArrayBase`, array dimension implementation tags, stride conversion, and access lowering through `get_access_info`.
  - This is a central file for array changes, but many consumers live elsewhere.

- `loopy/kernel/instruction.py`
  - Instruction classes, assignment/call/barrier semantics, dependency and access queries.

- `loopy/kernel/creation.py`
  - Kernel construction/parsing, temporary creation, shape inference, slice realization, and creation-time normalization.

- `loopy/kernel/tools.py`
  - Kernel analyses and utilities, array lookup/change helpers, shape guessing, alias classes, and graph-like queries.

- `loopy/kernel/function_interface.py`
  - Callable descriptors and specialization.
  - `ArrayArgDescriptor`, `InKernelCallable`, `CallableKernel`, subarray argument descriptor inference.

- `loopy/translation_unit.py`
  - Translation-unit representation, callable table, callable resolution, and specialization context.

### Expressions and polyhedral analysis

- `loopy/expression.py`
  - Expression-related checks and transformations.

- `loopy/symbolic.py`
  - Pymbolic mappers, affine conversion, access maps/ranges, `SubArrayRef`, dependency extraction, and overlap analysis.
  - Many analyses construct maps from instruction domains to array index tuples here.

- `loopy/isl_helpers.py`
  - Utilities around ISL/namedisl bounds, extrema, projections, and set/map manipulation.

- `loopy/types.py`, `loopy/typing.py`, `loopy/type_inference.py`
  - Loopy types, shared type aliases, and type inference.

### Transformations

- `loopy/transform/`
  - User-facing and internal kernel transformations.
  - Important modules include:
    - `data.py`: array-axis tags, temporary address-space setters, data-layout-related entry points.
    - `iname.py`: iname splitting/tagging and loop transformations.
    - `precompute.py`: prefetch/precompute transformations.
    - `privatize.py`: temporary duplication/private axes.
    - `realize_reduction.py`: reduction lowering.
    - `callable.py`: callable-kernel transformations.
    - `padding.py`, `buffer.py`, `array_buffer_map.py`: storage/layout changes.
    - `save.py`: save/reload across scheduling boundaries.
  - Many transform functions use `@for_each_kernel` so they operate on each callable kernel in a translation unit.

### Preprocessing, checks, scheduling, and code generation

- `loopy/preprocess.py`
  - Pre-codegen normalization.
  - Handles separate arrays, offsets/strides, ILP realization, temporary address-space inference, target preprocessing, and callable argument descriptor inference.
  - Ordering comments in this file are significant; do not reorder passes casually.

- `loopy/check.py`
  - Structural, bounds, access-ordering, race, callable, and pre-schedule checks.
  - Bounds checking and several address-space assumptions currently depend on tuple shapes and old address spaces.

- `loopy/schedule/`
  - Schedule generation and schedule-time dependency/barrier reasoning.
  - `schedule/tools.py` contains access-race logic and generated-subkernel argument decisions.
  - `schedule/__init__.py` contains dependency tracking and schedule generation machinery.

- `loopy/codegen/`
  - Target-independent code-generation state and control/loop/instruction lowering.

- `loopy/target/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inducer/loopy](https://github.com/inducer/loopy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

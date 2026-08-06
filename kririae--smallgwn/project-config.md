---
trigger: always_on
description: These instructions apply to the `smallgwn/` project tree.
---

# AGENTS.md

## Scope
These instructions apply to the `smallgwn/` project tree.

## Language and Build Baseline
- Use C++20 and CUDA 12+.
- CMake defaults `CMAKE_CUDA_ARCHITECTURES` to `native` unless overridden.
- Keep the project header-only at the library level.
- Repo build targets use strict warnings by default; do not add opt-in warning-profile toggles for
  benchmark/test coverage.
- When a build toggle enables a dependency-bearing path, every `find_package(...)` in that path
  must be `REQUIRED` and fail configure immediately on missing packages. Do not use `QUIET`,
  `*_FOUND` fallback branches, skip-on-missing behavior, or local search hacks/workarounds.

## File Naming Rules
- **`.cuh`**: Must be used if the file contains CUDA execution space specifiers, kernel launches,
  CUDA Runtime API calls, or includes other `.cuh` files.
- **`.hpp`**: Strictly for pure C++ CPU-side code; must compile without NVCC.

## Naming and API Rules
- Use `gwn::` namespace and `gwn_` prefix for public symbols.
- Default public index type is `std::uint32_t` unless explicitly overridden.
- Width-4 convenience aliases: `gwn_bvh4_object`, `gwn_bvh4_accessor`, and
  `gwn_bvh4_moment_<role>`.
- Moment types carry `Order` as a compile-time template parameter (after `Width`).
- Owning-object state query: unified `has_data()` predicate.
- Detail entrypoints use `_impl` suffix to avoid public/internal naming collisions.

## Formatting Rules
- Format with `clang-format` using the project `.clang-format` (LLVM-based).
  **MUST** run `clang-format` on every changed C++/CUDA file before committing.
- **Internal `#include` paths must always be relative**:
  - `include/gwn/`: bare names (`"gwn_bvh.cuh"`).
  - `include/gwn/detail/`: bare names for siblings, `../` for parent-level.
  - Never use the rooted `"gwn/..."` form inside library headers.

## Architecture & Design Rules

Four decisions generate most rules below. (1) Owning host objects pair with trivially copyable
device accessors: objects manage lifetime, accessors are the device-facing view. (2) Public host
entrypoints that can fail are `noexcept` and return `gwn_status`; detail host execution code throws
and is translated at the seam. (3) No lifetime operation or stream-binding transfer synchronizes
implicitly; allocation, use, and release remain stream-ordered. (4) Prefer exact contracts over
conservative ones (topology-exact stack bounds, closed-interval slab tests, staging-then-swap
replacement) so validity is checkable instead of assumed. When a situation is not covered by a
rule below, derive the answer from these four decisions.

### Public / Detail Split
- Public headers under `include/gwn/` expose the minimal API surface.
  Implementation lives in `include/gwn/detail/`; public headers may `#include` detail headers
  (required for header-only templates), but users should never `#include` detail headers directly.
- `gwn_utils.cuh` is the public foundation for template constraints, status values, stream binding,
  index conventions, and stream-ordered raw allocation. Exception adapters, span ownership, and
  kernel launch machinery live in `include/gwn/detail/`.

### Geometry
- SoA layout (`x/y/z`, `i0/i1/i2`).
- `gwn_geometry_object` owns only vertex positions and oriented triangle indices.
- `gwn_boundary_chain_object` stores the algebraic boundary of an oriented triangle-index chain.
  It is geometry-derived data, separate from BVH payloads and Taylor moments.
- The uploaded-geometry boundary builder is object-based; device point queries consume accessors.

### BVH
- `gwn_bvh_object` owns one canonical query structure: child-AoS bounds and references, primitive
  order, and leaf-ordered triangle records. Moment objects remain independent field-SoA payloads
  aligned to one BVH revision.
- Public API split: `gwn_bvh_build.cuh` builds the canonical BVH and `gwn_bvh_refit.cuh` refits
  geometry-derived BVH data or one moment order.
- `gwn_build_bvh` exposes H-PLOC by default and LBVH through `gwn_bvh_build_options`; H-PLOC's
  nearest-neighbor search radius is a runtime option in the inclusive range `[1, 8]`.
- BVH accessors store topology-exact `internal_stack_bound` and `packed_stack_bound` values computed
  during wide collapse. Host batch query launchers reject `StackCapacity` below the internal bound;
  packed ray traversal is selected only when the packed bound also fits.
- Taylor moment supports `Order=0/1/2`.
  Each `gwn_refit_bvh_moment<Order,...>` call does a full replace of the moment accessor.
- A successful BVH refit advances its revision and makes existing moment objects stale. Refit each
  required moment order explicitly before its next query.
- Accessors are trivially copyable snapshots of an object's device state. Any successful build,
  refit, clear, or destruction invalidates previously copied accessors. `revision` is the
  alignment identity: zero means no published queryable BVH state, and moment validity requires
  `bvh_revision == bvh.revision` in addition to structural checks.
- `accessor()` is mutable by design: it is the expert escape hatch for assembling custom
  accessors (tests rely on it). Any non-empty span installed through this path must be the base of a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kririae/smallgwn](https://github.com/kririae/smallgwn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

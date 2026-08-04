---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**IMPORTANT**: when running Julia, always run with `julia --project=docs` so that you have access to utility packages
for loading geometry, etc. etc.  If you get an error about a package not being found, try running `julia --project=docs -e 'using Pkg; Pkg.instantiate()'` first.

### Testing
Run all tests:
```bash
julia --project=. -e 'using Pkg; Pkg.test()'
```

To run a single test file in isolation, simply include it while in the test environment (test files are self-contained — each one does its own `using`/`import`):

```bash
julia --project=test -e 'include("test/methods/area.jl")'
```

### Testing Against Multiple Geometry Implementations

The **GeometryOpsTestHelpers/** subpackage provides `@test_implementations` and `@testset_implementations`. These macros run a test block once per geometry implementation (GeoInterface wrappers always; ArchGDAL, GeometryBasics, and LibGEOS are added to `GeometryOpsTestHelpers.TEST_MODULES` via package extensions when those packages are loaded). Variables prefixed with `$` are converted to each module's geometry type with `GeoInterface.convert`:

```julia
using GeometryOpsTestHelpers  # also import LibGEOS/ArchGDAL/etc. to activate their extensions

poly = GI.Polygon([[(0.0, 0.0), (1.0, 0.0), (1.0, 1.0), (0.0, 0.0)]])

@test_implementations GO.area($poly) == 0.5

@testset_implementations "Area" begin
    @test GO.area($poly) == 0.5
end
```

Prefer these macros when testing public API functions, so behavior is verified across all GeoInterface-compatible geometry libraries.

### Git Commit Style
Commit messages in this repository follow a simple, descriptive style:

- **Use imperative mood**: "Fix bug" not "Fixed bug" or "Fixes bug"
- **Start with a capital letter**: "Add feature" not "add feature"
- **Be concise but descriptive**: Explain what changed, not why (unless non-obvious)
- **No trailing periods**: Commit messages don't end with a period
- **Use backticks for code**: Reference functions/types with backticks like `smooth` or `TraitTarget`
- **No conventional commit prefixes**: Don't use "feat:", "fix:", "docs:", etc.

Examples from the project:
```
Fix type constraint in _smooth function
Add a `smooth` function 
Refactor tests to be a bit easier to parse
Tree based acceleration for polygon clipping / boolean ops
Bump version from 0.1.30 to 0.1.31
```

## High-Level Architecture

### Monorepo Structure
GeometryOps uses a monorepo structure, organized as a Julia workspace (see `[workspace]` in `Project.toml`; subpackages are resolved by path via `[sources]`):
- **GeometryOpsCore/**: Core abstractions, types, and primitive functions (`apply`, `applyreduce`, `flatten`, etc.)
- **GeometryOpsTestHelpers/**: Test utilities (`@test_implementations`, `@testset_implementations`) for running tests across multiple geometry implementations
- **src/**: Main package implementation
- **ext/**: Package extensions for optional dependencies (LibGEOS, Proj, TGGeometry, DataFrames, FlexiJoins, Makie)

### Core Abstractions

**GeoInterface Integration**: All functions work with any GeoInterface-compatible geometry. Dispatch is based on GeoInterface traits (PointTrait, LineStringTrait, PolygonTrait, etc.), not concrete types.

**Manifold System**: Operations can be performed on different manifolds:
- `Planar()`: Euclidean/Cartesian coordinates (default)
- `Spherical()`: Spherical coordinates on a unit sphere
- `Geodesic()`: Geodesic calculations on Earth (requires Proj extension)
- `AutoManifold()`: Automatically select appropriate manifold

Functions typically accept a manifold as the first argument:
```julia
area(Planar(), polygon)
area(Spherical(), polygon)
```

**Apply Framework**: The `apply` and `applyreduce` functions from GeometryOpsCore are the workhorses for geometry operations:
- `apply`: Applies a function to geometries matching a target trait, then reconstructs the geometry
- `applyreduce`: Applies a function and reduces the results (e.g., sum, min, max)
- `TraitTarget`: Specifies which geometry traits to target (e.g., `TraitTarget{GI.PointTrait}()`)

Example pattern:
```julia
applyreduce(WithTrait((trait, g) -> _area(T, trait, g)), +, _AREA_TARGETS, geom; threaded, init=zero(T))
```

### Code Organization Principles

1. **Literate Programming**: Source files use literate programming with documentation and examples at the top, followed by implementation. Examples should include visual plots using Makie/CairoMakie when appropriate.

2. **One File, One Job**: Each file should handle one semantic concept (e.g., `area.jl`, `distance.jl`, `centroid.jl`). Common utilities can be extracted to separate files.

3. **Public vs Internal**:
   - Public functions: Exported, documented, promise API stability
   - Internal functions: Prefixed with `_`, not exported, may have comments instead of docstrings

4. **File Structure Pattern** (see `src/methods/area.jl` or `src/methods/distance.jl`):
   ```julia
   # # Title
   export function_name

   #=
   ## What is [concept]?
   [Explanation with examples, plots]

   ## Implementation
   [Implementation notes]
   =#

   # Public API with docstring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuliaGeo/GeometryOps.jl](https://github.com/JuliaGeo/GeometryOps.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

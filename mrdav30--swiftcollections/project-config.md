---
trigger: always_on
description: SwiftCollections is a framework-agnostic .NET collection library for performance-sensitive code: simulations, games, spatial queries, deterministic runtimes, pooling-heavy systems, and tooling that needs predictable data-structure behavior.
---

# SwiftCollections Contributor Guide

## Purpose

SwiftCollections is a framework-agnostic .NET collection library for performance-sensitive code: simulations, games, spatial queries, deterministic runtimes, pooling-heavy systems, and tooling that needs predictable data-structure behavior.

The standard .NET collections remain the right default for ordinary application code. This repository exists for the places where hot-path cost, storage layout, dense iteration, deterministic hashing, pooling, or specialized integer-ID ownership justify a sharper collection.

Current priorities:

1. Prefer optimized, low time-complexity implementations. No band-aid solutions.
2. Preserve public API behavior and exception contracts unless a breaking change is intentional.
3. Keep hot paths allocation-conscious and benchmarkable.
4. Keep the core library engine-agnostic; Unity-specific packaging belongs in the separate Unity repository.
5. Maintain high test coverage, especially for new public APIs, serialization state, and edge-case branches.
6. Keep standard and lean package variants aligned.

## Start Here

Read these before making non-trivial changes:

1. `README.md` for the public front door and package orientation.
2. `docs/OVERVIEW.md` for the collection, spatial query, serialization, and diagnostics map.
3. `SwiftCollections.slnx` and the relevant `*.csproj` files. They are the source of truth for compiled projects, package shape, and target frameworks.
4. The source folder under `src/SwiftCollections` or `src/SwiftCollections.FixedMathSharp` that owns the change.
5. The matching tests under `tests/SwiftCollections.Tests` or `tests/SwiftCollections.FixedMathSharp.Tests`.
6. Benchmarks under `tests/SwiftCollections.Benchmarks` when a change touches hot-path behavior or a performance claim.
7. `docs/complexity-exceptions.md` before refactoring high-complexity methods or adding a new intentional exception.

## Source Of Truth

When docs, workflows, and project files disagree, trust the project files and source code first, then update the docs.

Current compiled projects:

| Project | Path | Target Frameworks |
| --- | --- | --- |
| Core library | `src/SwiftCollections/SwiftCollections.csproj` | `netstandard2.1;net8.0` |
| FixedMathSharp companion | `src/SwiftCollections.FixedMathSharp/SwiftCollections.FixedMathSharp.csproj` | `netstandard2.1;net8.0` |
| Core tests | `tests/SwiftCollections.Tests/SwiftCollections.Tests.csproj` | `net8.0` |
| FixedMathSharp tests | `tests/SwiftCollections.FixedMathSharp.Tests/SwiftCollections.FixedMathSharp.Tests.csproj` | `net8.0` |
| Benchmarks | `tests/SwiftCollections.Benchmarks/SwiftCollections.Benchmarks.csproj` | `net8` |

Keep these aligned whenever behavior, public API, package variants, or workflow expectations change:

- `README.md`
- `docs/OVERVIEW.md`
- `docs/complexity-exceptions.md`
- relevant tests and benchmarks
- `.github/workflows/build-and-test.yml`
- `.github/workflows/coverage.yml`
- `.github/workflows/publish-nuget.yml`

## Repository Map

| Path | Purpose | Notes |
| --- | --- | --- |
| `src/SwiftCollections/Collection` | Core collection types | Includes dictionary, hash set, list, queue, stack, bucket, packed set, sparse set, and sparse map. |
| `src/SwiftCollections/Dimension` | Flat 2D/3D arrays and typed array helpers | Preserve index math and bounds behavior. |
| `src/SwiftCollections/EqualityComparer` | SwiftCollections comparers | String default comparers are deterministic-oriented. |
| `src/SwiftCollections/Observable` | Observable collection variants | Prefer for tooling/host-facing notifications, not simulation hot paths without evidence. |
| `src/SwiftCollections/Pool` | Object, array, and collection pools | Watch ownership and dispose/release contracts. |
| `src/SwiftCollections/Query` | BVH, spatial hash, octree, bounds, and query helpers | Compiled in the core package. Keep mutable indexes single-owner unless synchronized externally. |
| `src/SwiftCollections/Serialization` | State structs and JSON/state converter support | Build after changing state shape or serialization constructors. |
| `src/SwiftCollections/Support` | Compatibility shims | Includes MemoryPack and interpolated-string-handler shims for target/package variants. |
| `src/SwiftCollections/Utility` | Shared helpers, hashing, diagnostics, extensions | Preserve helper exception contracts. |
| `src/SwiftCollections.FixedMathSharp` | Fixed-point spatial query companion | Depends on FixedMathSharp or FixedMathSharp.Lean based on package variant. |
| `tests/SwiftCollections.Tests` | xUnit v3 core tests | Mirror source areas. |
| `tests/SwiftCollections.FixedMathSharp.Tests` | FixedMathSharp companion tests | Uses the shared coverage runsettings from the core test project. |
| `tests/SwiftCollections.Benchmarks` | BenchmarkDotNet benchmarks | Alias-based runner supports `list`, `dictionary`, `query`, `all`, and other selections. |
| `.assets/scripts` | Windows-oriented release/version helpers | Assumes GitVersion tooling. |

Ignore generated output when reviewing structure:

- `bin/`
- `obj/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrdav30/SwiftCollections](https://github.com/mrdav30/SwiftCollections) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->

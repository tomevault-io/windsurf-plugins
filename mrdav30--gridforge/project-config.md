---
trigger: always_on
description: This document is for both human contributors and AI coding agents working in this repository. It is intentionally practical: it describes what the project is, how the codebase is organized, which invariants matter, and how to make changes without breaking deterministic grid behavior.
---

# GridForge Agent Guide

This document is for both human contributors and AI coding agents working in this repository. It is intentionally practical: it describes what the project is, how the codebase is organized, which invariants matter, and how to make changes without breaking deterministic grid behavior.

## Project Summary

GridForge is a deterministic voxel-grid library for spatial partitioning, simulation, and game-development use cases. The core library is framework-agnostic and centers on:

- explicit `GridWorld` ownership and world-scoped grid registration
- voxelized world-space bounds
- scan-cell overlays for fast spatial queries
- obstacle and occupant tracking
- deterministic fixed-point math through `FixedMathSharp`
- allocation-conscious collections and pools through `SwiftCollections`

The repository currently contains one library project and two validation projects:

- `src/GridForge` - main library
- `tests/GridForge.Tests` - xUnit test suite
- `tests/GridForge.Benchmarks` - BenchmarkDotNet performance and allocation benchmarks

## Technology and Build Facts

- **Language:** C# 11
- **Library target frameworks:** `netstandard2.1`, `net8.0`
- **Validation target frameworks:** `net8.0`
- **Test framework:** xUnit v3
- **Benchmark framework:** BenchmarkDotNet
- **Main dependencies:** `FixedMathSharp` `2.1.0`, `SwiftCollections` `2.0.0`
- **Build behavior:** `dotnet build` on the library also produces NuGet packages because `GeneratePackageOnBuild` is enabled in `src/GridForge/GridForge.csproj`
- **CI:** runs on Ubuntu and Windows via `.github/workflows/build-and-test.yml`
- **Versioning:** CI uses GitVersion; local builds without GitVersion fall back to version `0.0.0`

## Repository Layout

- `README.md` - external-facing project overview and usage examples
- `GridForge.slnx` - solution entry point
- `src/GridForge/GridForge.csproj` - library project configuration and package metadata
- `src/GridForge/Configuration` - grid configuration types and bounds identity
- `src/GridForge/Grids` - core grid, voxel, scan-cell, manager, and pooling logic
- `src/GridForge/Spatial` - occupant, partition, and index abstractions
- `src/GridForge/Blockers` - area/blocking abstractions built on top of grid coverage
- `src/GridForge/Utility` - tracing and logging helpers
- `tests/GridForge.Tests` - unit tests organized by subsystem
- `tests/GridForge.Benchmarks` - benchmark scenarios for pooling, tracing, caching, and registration performance
- `.github/workflows` - CI and release automation
- `.assets/scripts` - PowerShell helpers for versioned build and release packaging

Ignore these when reading or editing unless the task is specifically about build outputs or IDE state:

- `.vs`
- `bin`
- `obj`
- `TestResults`

## Architecture At A Glance

### Core Types

- **`GridWorld`:** primary runtime owner for one world's mutable state: setup values, active grids, spatial hash, world-space lookups, and world-level events.
- **`VoxelGrid`:** represents a single configured grid with voxels, scan cells, neighbor relationships, occupancy state, and versioning.
- **`Voxel`:** holds world position, grid indices, obstacle state, occupancy state, partition attachments, and cached neighbor data.
- **`ScanCell`:** secondary overlay used to accelerate neighborhood and area queries over voxels.
- **`GridTracer`:** converts lines and bounding regions into covered voxel sets inside an explicit `GridWorld`.
- **`BoundsBlocker` and `Blocker`:** apply or remove obstacle state across covered voxels returned by tracing logic.
- **`PartitionProvider`, `IVoxelPartition`, and `IVoxelOccupant`:** extension points for custom metadata and occupant systems.

### Important Design Characteristics

- Deterministic math matters. The library uses `Fixed64`, `Vector2d`, and `Vector3d` from `FixedMathSharp`.
- Grid state is world-scoped. Most runtime APIs should be anchored to an explicit `GridWorld`.
- Bounds are snapped to voxel size. Many APIs normalize or snap incoming coordinates.
- Object pooling is used heavily for grids, voxels, scan cells, arrays, and temporary collections.
- Performance-sensitive code favors explicit control flow over abstraction-heavy patterns.

## Critical Invariants

Treat the following as core rules of the system:

- Create a `GridWorld` before using world-scoped grid APIs.
- Dispose a `GridWorld` or call `Reset()` when a test or tool run needs a clean world state.
- Use fixed-point types for grid math. Avoid introducing `float` or `double` into core simulation logic unless there is a clear boundary conversion reason.
- Assume bounds and positions may be snapped to the configured voxel size. When debugging odd query results, check snapped values first.
- Respect pooling. If a type or collection comes from a pool, verify whether it is safe to retain beyond the immediate operation.
- Preserve deterministic behavior across target frameworks. If a change behaves differently between `netstandard2.1` and `net8.0`, treat that as a bug.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrdav30/GridForge](https://github.com/mrdav30/GridForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->

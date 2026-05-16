---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CoursePointer converts GPX routes/tracks and waypoints into Garmin FIT course files with course points. It solves the problem of making waypoints from third-party apps (Gaia GPS, Ride with GPS) appear correctly in the "Up Ahead" feature on Garmin devices by computing precise geodesic distances where waypoints intercept routes.

**Key Technologies**: Rust, GeographicLib (C++ FFI), WebAssembly, Python (integration tests)

## Essential Commands

### Building and Running

```bash
# Initial setup - import GeographicLib submodule (required after cloning)
git submodule update --init

# Build and run CLI
cargo run -F cli -- <CLI_ARGS>

# Build CLI binary only
cargo build -F cli

# Build library (no features needed)
cargo build
```

### Testing

```bash
# Run Rust tests
cargo test

# Run Python integration tests (primary test suite)
uv run --package integration pytest

# Run specific test file
uv run --package integration pytest integration/tests/test_name.py

# Run specific test function
uv run --package integration pytest integration/tests/test_name.py::test_function_name
```

The Python integration tests use the `garmin-fit-sdk` as a reference implementation to validate FIT encoding. Some tests use the `integration-stub` binary (in devtools package) for low-level FIT encoding verification.

### Formatting and Linting

```bash
# Rust formatting (requires nightly for unstable features)
rustup toolchain install --profile minimal -c rustfmt nightly
cargo +nightly fmt

# Python formatting and linting
ruff check
ruff format
```

### Web Application

```bash
cd web
pnpm install
pnpm build
pnpm dev
```

See `.github/actions/web/action.yml` for web testing and linting commands.

### Profiling

```bash
# Set environment variable before running cargo-flamegraph
CARGO_PROFILE_RELEASE_DEBUG=true cargo flamegraph <args>

# Or enable debug symbols by uncommenting in Cargo.toml:
# [profile.release]
# debug = "line-tables-only"
```

Set log level `-L debug` for tracing summaries (has measurement overhead on dev builds).

## Architecture

### High-Level Data Flow

```
GPX Input → GPX Parser → CourseSetBuilder → Course + CoursePoints → FIT Encoder → FIT Output
            (gpx.rs)     (course.rs)         (course.rs)              (fit.rs)
```

### Core Modules

- **`lib.rs`**: High-level API. Main entry points: `convert_gpx_to_fit()`, `read_gpx()`, `write_fit_course()`
- **`course.rs`**: Course building and computation. `CourseSetBuilder` assembles routes/waypoints, computes which waypoints intercept routes, and calculates their distances
- **`gpx.rs`**: Streaming XML parser for GPX files. Identifies route creator (Gaia GPS, Ride with GPS, etc.)
- **`fit.rs`**: FIT binary format encoding. Uses Garmin FIT Profile Version 21158
- **`algorithm.rs`**: Geodesic interception problem solver using Karney's iterative gnomonic projection
- **`geographic.rs`**: FFI wrapper for GeographicLib C++ library (accurate WGS84 ellipsoidal calculations)
- **`point_type.rs`**: Maps third-party POI symbols to Garmin course point types (see docs/point_types.md)
- **`types.rs`**: Core geographic types with invariants (`GeoPoint`, `GeoSegment`)
- **`measure.rs`**: Dimensional analysis types (leverages `dimensioned` crate for type-safe units)

### Key Types

- **`GeoPoint`**: Geographic point with lat/lon/elevation, maintains type invariants
- **`Course`**: Route with computed distances and associated course points
- **`CourseSet`**: Collection of courses with waypoints (typically 1 course per GPX file)
- **`CoursePoint`**: Waypoint that has been matched to a course with computed distance
- **`CoursePointType`**: Enum of Garmin FIT course point types (food, water, shelter, danger, etc.)

### GeographicLib Integration

CoursePointer depends on GeographicLib (C++ library, included as git submodule at `geographiclib/`) for geodesic calculations. The Rust code interfaces via FFI through `src/shim.cpp` and `src/shim_embind.cpp`.

**Important**: After updating the GeographicLib submodule, regenerate `include/GeographicLib/Config.h`:
```bash
cd geographiclib
mkdir BUILD && cd BUILD
cmake .. -DBUILD_SHARED_LIBS=OFF
cp include/GeographicLib/Config.h ../../include/GeographicLib/
```

### Feature Flags

- `cli`: Enables CLI binary dependencies (clap, sys-locale, tracing-subscriber, etc.)
- `rayon`: Parallel course point computation (default, significant performance improvement)
- `full-geolib`: Build all GeographicLib sources (normally a subset is built; use for FFI experiments)
- `jsffi`: WebAssembly build with wasm-bindgen (replaces C FFI for web builds)

### Workspace Structure

This is a Cargo workspace with three members:
- Root: Main library and CLI (`coursepointer` crate)
- `devtools`: Contains `integration-stub` binary for bootstrapping FIT encoding tests
- `web/coursepointer-wasm`: WASM compilation of core logic for web app

### POI Type Mapping

When processing GPX from Gaia GPS or Ride with GPS, CoursePointer automatically maps waypoint symbols to meaningful Garmin course point types. See `docs/point_types.md` for comprehensive mapping tables and device compatibility.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mshroyer/coursepointer](https://github.com/mshroyer/coursepointer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->

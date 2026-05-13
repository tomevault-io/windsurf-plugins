---
trigger: always_on
description: GeoParquet → PMTiles converter in Rust. Library-first design with CLI and Python bindings.
---

# gpq-tiles - Claude Code Instructions

## Project Overview

GeoParquet → PMTiles converter in Rust. Library-first design with CLI and Python bindings.

**Goal:** Faster than tippecanoe for typical GeoParquet workflows, with native Arrow integration.

## Critical Constraints

### 1. Test-Driven Development (TDD) is MANDATORY

Every feature follows: **failing test → implementation → refactor**

```bash
cargo test --package gpq-tiles-core <module> -- --nocapture  # Verify red
# Implement
cargo test --package gpq-tiles-core <module> -- --nocapture  # Verify green
git commit -m "feat: implement X (TDD green)"
```

### 2. Arrow/GeoArrow: Columnar I/O, Not Zero-Copy Geometry Processing

**Arrow gives us efficient columnar I/O and streaming, but geometry operations require `geo::Geometry` conversion.**

What we GET from Arrow/GeoArrow:
- Columnar decoding (only geometry column parsed, properties lazy-loaded)
- Row-group streaming (memory = O(row_group), not O(file))
- No double-copy (Arrow → geo directly, not Arrow → WKB → geo)

What we DON'T get (yet):
- Zero-copy clipping (geo::BooleanOps requires owned `geo::Polygon`)
- Zero-copy simplification (geo::Simplify requires owned `geo::LineString`)

DO:
```rust
// Iterate geometries within Arrow batch, convert only what's needed
for batch in reader {
    let geom_col = batch.column(geom_idx);
    let geom_array = geoarrow::array::from_arrow_array(geom_col, geom_field)?;
    for geom in geom_array.iter() {
        let geo_geom: geo::Geometry = geom.try_to_geometry()?;  // Conversion needed for clipping
        let clipped = clip_geometry(&geo_geom, &tile_bounds)?;
        // Process immediately, don't accumulate all features
    }
}
```

DO NOT:
```rust
// WRONG: Deserializing to WKB first defeats Arrow's columnar benefits
let geom: geo::Geometry = geozero::wkb::Wkb(wkb.to_vec()).to_geo();
```

### 3. Reference Implementations (CRITICAL)

**All algorithms MUST match tippecanoe behavior as closely as possible.**

- **tippecanoe** (https://github.com/felt/tippecanoe) - PRIMARY reference
- **planetiler** (https://github.com/onthegomap/planetiler) - Secondary reference

**When deviating from tippecanoe:**
```rust
// DIVERGENCE FROM TIPPECANOE: [reason]
// Tippecanoe does X (see tile.cpp:L312)
// We do Y because [Rust limitation / performance / etc.]
```

Document all divergences in `context/ARCHITECTURE.md`.

### 4. Test Execution: Targeted Tests Only

**Tests are SLOW.** The full suite runs integration tests with real parquet I/O, full pipeline execution, and nested parallelism (cargo test threads × Rayon threads × I/O threads).

**NEVER run the full test suite.** Always run targeted tests:

```bash
# GOOD: Run specific test
cargo test --package gpq-tiles-core batch_processor::tests::test_specific_thing -- --nocapture

# GOOD: Run tests in a specific module
cargo test --package gpq-tiles-core covering:: -- --nocapture

# BAD: Runs everything, takes forever
cargo test
cargo test --package gpq-tiles-core
```

**When to skip tests entirely:**
- Formatting fixes (`cargo fmt`)
- Import cleanup
- Documentation changes
- Changes already verified by `cargo check` or `cargo build`

**Use `cargo check` liberally** — it's fast and catches most errors without running tests.

## Architecture

```
crates/
├── core/     # ALL tiling logic lives here
├── cli/      # Thin wrapper: args → core::convert()
└── python/   # pyo3 bindings → core
```

**Library-first:** CLI and Python are thin consumers. Never put logic in CLI/Python that belongs in core.

## Key Types

```rust
TileCoord { x: u32, y: u32, z: u8 }           // Tile coordinates (Web Mercator)
TileBounds { lng_min, lat_min, lng_max, lat_max }  // Geographic bounds
TilerConfig { min_zoom, max_zoom, ... }       // Pipeline configuration
```

## Commands

```bash
cargo check                   # Fast compile check (use liberally)
cargo build                   # Build debug
cargo build --release         # Build release
cargo fmt --all               # Format (required before commit)

# Tests - ALWAYS targeted (see Critical Constraint #4)
cargo test --package gpq-tiles-core <module>::<test> -- --nocapture

# Benchmarks (slow - only run when needed)
cargo bench --package gpq-tiles-core --bench <name> -- "<filter>"

# Run the CLI
cargo run --package gpq-tiles -- input.parquet output.pmtiles
```

## CRITICAL: Always Use UV for Python

**NEVER use bare `python`, `pip`, or `poetry`.** Always prefix with `uv run`:

```bash
cd crates/python && uv sync              # Install/sync deps
cd crates/python && uv run pytest        # Run tests
cd crates/python && uv run python script.py  # Run any Python script
```

This ensures consistent environments and avoids "module not found" errors.

## Git Workflow

### Branch Protection

**The `main` branch is protected.** You cannot push directly to main.

All changes must go through pull requests:
```bash
# Create a feature branch
git checkout -b fix/my-fix
git push -u origin fix/my-fix

# Create PR
gh pr create --title "fix: description" --body "..."
```

### DO NOT

- Push directly to `main` (will be rejected)
- Force push to any shared branch
- Merge your own PRs without CI passing

## Common Pitfalls


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geoparquet-io/gpq-tiles](https://github.com/geoparquet-io/gpq-tiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

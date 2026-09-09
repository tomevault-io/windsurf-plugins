---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`rtz` is a self-contained geo lookup library / binary / server (also compiles to WASM/WASI) that
resolves a `(lng, lat)` pair to a timezone or administrative region using the **Natural Earth
(NED)** and **OpenStreetMap (OSM)** datasets. The headline trick: dataset geometry is baked into
the binary as `bincode` and memory-mapped at runtime, so lookups need no network, no files, and no
allocation on the hot path.

## Commands

The toolchain is **pinned to stable `1.93`** via `rust-toolchain.toml`; don't assume newer. Build,
test, and lint all run on stable — only coverage (`cargo make codecov`) needs nightly, for the
nightly-gated `#[coverage(off)]` exclusions. A `Makefile.toml` (cargo-make) wraps the canonical
commands (`cargo make test` / `clippy` / `codecov`); the raw-cargo equivalents below also work.

```bash
# Test (the default feature set is tiny; most tests need datasets + server)
cargo test --features web            # what CI runs (web pulls in `full`)
cargo test --features full --features web   # full local test sweep (DEVELOPMENT.md)
cargo nextest run --features web     # nextest is preferred if installed; CI uses it

# Run a single test
cargo test --features web can_get_osm_timezone_v1
cargo nextest run --features web -E 'test(can_verify_lookup_assisted_accuracy)'

# Lint — every public item needs a doc comment (`missing_docs` is warn-as-error here)
cargo clippy --features web --all-targets

# Bench (criterion)
cargo bench --features web

# Run the CLI / server
cargo run --features full -- ned tz "-87.62,41.88"
cargo run --features web  -- serve            # Axum server, default port from config

# WASM/NPM build (see DEVELOPMENT.md for the full publish flow)
wasm-pack build --target web --no-default-features \
  --features tz-osm --features tz-ned --features self-contained --features wasm --features extrasimplified
```

## Workspace layout

Three crates. `rtz-build` is **not** a workspace member (the root `[workspace]` lists only
`rtz-core` and `rtz`); it is pulled in solely as `rtz`'s `[build-dependencies]` by path. All three
version independently and are published separately (`cargo publish -p rtz-core`, then `-p
rtz-build`, then `-p rtz` — bump the path-dep versions in `Cargo.toml` first).

- **`rtz-core`** — data types (`NedTimezone`, `OsmTimezone`, `OsmAdmin`), GeoJSON→struct
  conversion, the grid-cache generation logic, and all `bincode` `Encode`/`Decode` impls. This is
  the crate the build script calls into. It has no async/CLI/web surface.
- **`rtz-build`** — a thin build-script driver. Its `main()` is invoked from `rtz/build.rs` and,
  when `self-contained` is on, generates the `.bincode` asset files by calling
  `rtz_core::geo::shared::generate_bincodes`.
- **`rtz`** — the library (`rtzlib`), the `rtz` binary, the Axum web server, and the WASM bindings.
  Adds the runtime lookup traits and the memory-mapped decode path.

## Core architecture

### Two-phase data pipeline (build time → runtime)

1. **Build time** (`rtz-build`, only with `self-contained`): download or read the source GeoJSON,
   convert to `ConcreteVec<T>`, simplify geometry (Visvalingam–Whyatt), and write two bincode files
   per dataset into `rtz/assets/`: an **items** blob (the geometries/properties) and a **lookup**
   blob (the grid cache). The build script early-returns if the assets already exist unless the
   `force-rebuild` feature is set.
2. **Runtime** (`rtz`): each dataset type includes those bytes via `include_bytes_aligned!` and
   lazily `borrow_decode`s them into a `OnceLock`-backed static on first lookup.

The **grid cache** is the key perf idea: the world is bucketed into 1°×1° cells keyed by
`RoundLngLat = (i16, i16)`, each mapping to the list of geometry ids whose bounding box intersects
that cell. A lookup floors `(lng, lat)` to its cell, gets the candidate ids, and only then runs the
expensive point-in-polygon `contains` check on that short list (~96× faster average vs. scanning
all geometries — see `lookup_slow`, kept for correctness tests). `NedTimezone::lookup` further
short-circuits when a cell has exactly one candidate and the point isn't near the poles/antimeridian.

### Trait stack

Behavior is layered so helpers generalize across all three dataset types:

- `rtz-core`: `HasGeometry` + `HasProperties` (+ blanket `ToGeoJson*`). A dataset type is just a
  struct implementing these plus the bincode codecs.
- `rtz`: `HasItemData` (get the `&'static ConcreteVec<Self>`), `HasLookupData` (get the
  `&'static HashMap<RoundLngLat, Lookup>`), and `CanPerformGeoLookup` (the public `lookup`,
  `lookup_slow`, `memory_data_to_geojson`). This trait is the primary library entry point,
  re-exported at the crate root alongside `NedTimezone`/`OsmTimezone`/`OsmAdmin`.

### Zero-copy decode (the `unsafe` you'll see)

With the default (borrowed) decode, geometry `Vec`s and strings are reconstructed **directly over
the embedded binary bytes** via `Vec::from_raw_parts` on the aligned slice, avoiding a full copy of
the (up to ~46 MB) asset into the heap. This is why `bincode` is configured `Fixint`/`legacy` with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twitchax/rtz](https://github.com/twitchax/rtz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

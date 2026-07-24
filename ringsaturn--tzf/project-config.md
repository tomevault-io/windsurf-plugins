---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TZF is a high-performance timezone finder library for Go that determines the timezone for given latitude and longitude coordinates. The project is designed for geospatial services like weather forecast APIs where fast timezone lookups are critical.

## Development Commands

```bash
make fmt        # gofmt all packages
make test       # golangci-lint + go test -v with coverage
make cover      # test + open coverage HTML
make bench      # run benchmarks
make pb         # regenerate protobuf Go bindings via buf
```

Run a single test:
```bash
go test -v -run TestName ./internal/topology/...
```

Key tools required: `golangci-lint`, `buf` (proto generation), `go-licenses`.

## Core Architecture

### Finder Types (`tzf.go`, `tzf_fuzzy.go`, `tzf_default_finder.go`)

Three finder implementations share the interface in `f.go`:

| Finder | Mechanism | Memory | Speed |
|--------|-----------|--------|-------|
| `Finder` | Polygon point-in-polygon + grid index | ~30MB lite / ~150MB full | moderate |
| `FuzzyFinder` | Pre-indexed map tiles | ~2.4MB | fastest |
| `DefaultFinder` | FuzzyFinder first, Finder fallback (±0.02°) | ~32MB | fast |

Memory figures from `go run ./internal/bench-memory` (retained heap after GC).

`Finder` is a plain exported struct whose storage-generic internals hide behind
the unexported `finderCore` interface → `finderImpl[T geom.Coord]`. Data loaded
via `NewFinderFromPB` stores float64 degree coordinates; data loaded via
`NewFinderFromCompressedTopo` keeps the 1e5-scaled int32 polyline grid
(`geom.I32Polygon`, half the per-point memory, slightly more precise than the
old float32 protobuf round-trip). One interface dispatch per query; everything
below it is monomorphised. `internal/cmd/i32compare` cross-checks the two
storage paths on the bundled dataset.

### Data Pipeline

```
Raw GeoJSON (timezone-boundary-builder)
  └─ cmd/geojson2tzpb
       └─ combined-with-oceans.bin                (~92MB, Timezones, full precision)
            │
            ├─ cmd/reducetzpb -topology=true
            │    └─ combined-with-oceans.topology.bin   (~13MB, Timezones, topology-aware D-P simplified)
            │         ├─ cmd/deduplicatetzpb
            │         │    └─ combined-with-oceans.topology.topo.bin   (~10MB, TopoTimezones)
            │         │         └─ cmd/compresstopotzpb
            │         │              └─ combined-with-oceans.topology.compress.topo.bin  (~5.4MB) ← lite embedded
            │         └─ cmd/preindextzpb
            │              └─ combined-with-oceans.topology.preindex.bin (~2MB) ← preindex embedded
            │
            └─ cmd/deduplicatetzpb
                 └─ combined-with-oceans.topo.bin        (~52MB, TopoTimezones)
                      └─ cmd/compresstopotzpb
                           └─ combined-with-oceans.compress.topo.bin   (~17MB) ← full embedded
```

All three embedded files live in `github.com/ringsaturn/tzf-dist` (Go module, `data` branch). The `DefaultFinder` uses `topology.compress.topo.bin` + `topology.preindex.bin`; `NewFullFinder` uses `compress.topo.bin` + `topology.preindex.bin`. Versions must match between Finder and FuzzyFinder.

### Protobuf Schema (`pb/tzf/v1/tzinfo.proto`)

Key message families:
- **`Timezones` / `Timezone` / `Polygon` / `Point`** — flat polygon format used by all finders
- **`CompressedTimezones`** — polyline-encoded coordinates (existing lite format)
- **`TopoTimezones` / `SharedEdge` / `RingSegment`** — shared-edge deduplication format; rings are sequences of segment references pointing into a global edge library
- **`CompressedTopoTimezones`** — `TopoTimezones` with polyline-encoded point sequences

Regenerate after proto changes: `make pb` (runs `buf generate`).

### `internal/topology` Package

The topology-aware simplification engine. Key files:

- **`topology.go`** — `DoWithStats(input, epsilon)` is the main entry point. Pipeline: normalize coordinates → fix winding order → remove zero-length edges → snap T-junction vertices → collect rings + edge/vertex indices → mark shared edges → mark fixed vertices → simplify each ring using Douglas-Peucker with a shared-segment cache → validate fallbacks.
- **`dedup.go`** — `BuildTopoTimezones` / `DecodeTopoTimezones`: converts flat `Timezones` into the `TopoTimezones` shared-edge format. Uses `markFixedVerticesForDedup` (stricter than the simplification variant) to split rings at shared/non-shared boundaries.
- **`validate.go`** — `Validate` / `MustValidateForReduction`: geometry checks (winding, closure, self-intersection, zero-length edges). `ReductionValidateOptions` disables same-direction shared edge checks for disputed-territory data.

**Critical invariants:**
- `normalizeWindings` must be called *before* `snapVertices` and topology analysis so adjacent rings traverse shared boundaries in opposite directions.
- `removeZeroLengthEdges` must run before `collectRings`; source data can contain rings where adjacent (or wrap-around) points are identical, which breaks shared-edge detection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ringsaturn/tzf](https://github.com/ringsaturn/tzf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

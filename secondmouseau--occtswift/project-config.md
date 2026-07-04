---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Summary

OCCTSwift is a comprehensive Swift wrapper for OpenCASCADE Technology (OCCT) 8.0.0 (GA). It exposes B-Rep solid modeling capabilities to Swift for macOS (arm64, v12+) and iOS (arm64, v15+) via a three-layer architecture: Swift public API → Objective-C++ bridge (C functions) → OCCT C++ library. Uses Swift 6 language mode (strict concurrency).

## Build & Test Commands

```bash
swift build                          # Build the package
swift build --target OCCTThreadTests # Focused compile: just one domain's tests (~3s) — see "Test Layout"
swift test                           # Run all tests (~3900 tests across per-domain targets)
swift test --filter "Issue187"       # Run suites whose struct name matches (matches the type, not @Suite title)
swift run OCCTTest                   # Run test executable
```

### Compile a Ground Truth C++ Test

```bash
clang++ -std=c++17 -ObjC++ -w \
  -I"Libraries/OCCT.xcframework/macos-arm64/Headers" \
  -L"Libraries/OCCT.xcframework/macos-arm64" \
  -lOCCT-macos -framework Foundation -framework AppKit -lz -lc++ \
  /tmp/occt_vXX_test.mm -o /tmp/occt_vXX_test
/tmp/occt_vXX_test
```

### Verify OCCT Symbols

```bash
nm -C Libraries/OCCT.xcframework/macos-arm64/libOCCT-macos.a 2>/dev/null | grep "ClassName" | head -5
```

### OCCT Reference Docs (context7)

OCCT's developer overview / user guides (the `dev.opencascade.org/doc/overview` content,
generated from the repo's `dox/` guides) plus the wiki and headers are available via context7
as **`/open-cascade-sas/occt`**. Query it when wrapping new ops or checking C++ API usage
(e.g. `BRepAlgoAPI` options, `ThruSections`, healing) — it complements `/audit-occt` and the
header-analyzer agent. **Caveats:** context7's snapshot is the **occt-7.9** branch (+ some
`master`), while this project pins **OCCT 8.0.0** — for version-sensitive details, the pinned
headers in `Libraries/OCCT.xcframework/.../Headers` are the source of truth. It documents the
upstream C++ API the bridge wraps, not the Swift surface.

## Architecture

```
Sources/OCCTSwift/          Swift public API (Shape, Wire, Surface, Face, Edge, Curve3D, Mesh, etc.)
Sources/OCCTBridge/include/ C function declarations (single file: OCCTBridge.h)
Sources/OCCTBridge/src/     Objective-C++ implementations (single file: OCCTBridge.mm)
Libraries/OCCT.xcframework  Pre-built OCCT static library (arm64 macOS/iOS)
Tests/OCCT<Domain>Tests/    Per-domain Swift Testing targets (see "Test Layout")
Scripts/build-occt.sh       Builds OCCT.xcframework from source
```

### Handle-Based Memory Management

Opaque handle types (`OCCTShapeRef`, `OCCTWireRef`, `OCCTFaceRef`, `OCCTEdgeRef`, `OCCTMeshRef`) are typedef'd pointers. Swift classes wrap these handles and call the corresponding `Release` function in `deinit`. Every bridge function that creates an OCCT object must have a matching `Release` function.

### Adding a New Wrapped Operation

1. **Bridge header** (`OCCTBridge.h`): Add C function declaration
2. **Bridge impl** (`OCCTBridge.mm`): Add Objective-C++ implementation calling OCCT C++ API
3. **Swift wrapper** (appropriate `.swift` file): Add public method/static factory
4. **Test**: Add `@Suite`/`@Test` to the matching `Tests/OCCT<Domain>Tests/` target (see "Test Layout")

## Naming Conventions

- Bridge functions: `OCCTShape...`, `OCCTWire...`, `OCCTFace...`, `OCCTEdge...`
- Wire-to-shape conversion: `OCCTShapeFromWire()` (NOT `OCCTWireToShape`)
- Check enum values: `OCCTCheckNoError` (NOT `OCCTCheckStatusNoError`)
- `vertices()` is a method, not a property
- Swift factory methods are static: `Shape.box()`, `Wire.rectangle()`
- Fallible operations return optionals, not force-unwrapped values

## Test Layout

Tests are split into **per-domain test targets** (one Swift module each) so editing/compiling
one domain never recompiles the rest. The old 50k-line `ShapeTests.swift` monolith was split by
suite into these targets (each `Tests/OCCT<Domain>Tests/`, declared in `Package.swift`):

`Analysis`, `Curve`, `Drawing`, `Foundation`, `Geom2d`, `IO`, `Integration`, `Math`, `Mesh`,
`Misc`, `Modeling`, `ShapeHealing`, `Stress`, `Surface`, `Thread`, `TopologyGraph`, `Topology`, `XCAF`.

- **Add a new suite** to the domain target that best matches it (e.g. a fillet suite → `OCCTModelingTests`,
  a `Curve2D` suite → `OCCTGeom2dTests`). If nothing fits, use `OCCTMiscTests`. Each target is a separate
  module with its own `@testable import OCCTSwift`; the only shared helper is `SIMD3.normalized` (redefine
  it in the target if needed).
- **Focused compile** (the point of the split): `swift build --target OCCTThreadTests` type-checks just
  that module in ~3 s — never touches the other domains.
- **Focused run:** `swift test --filter <StructName>` (the filter matches the test *struct* name, e.g.
  `Issue187`, not the `@Suite("...")` display string). `swift test` still runs everything.
- The full suite remains prone to the non-deterministic NCollection arm64 SEGV under parallel execution
  (see Known OCCT Bugs); a single domain target rarely trips it.

## Test Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SecondMouseAU/OCCTSwift](https://github.com/SecondMouseAU/OCCTSwift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->

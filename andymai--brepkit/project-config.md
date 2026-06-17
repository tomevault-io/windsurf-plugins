---
trigger: always_on
description: brepkit is the B-Rep modeling engine behind brepjs. It provides the computational
---

# brepkit — Project Guidelines

brepkit is the B-Rep modeling engine behind brepjs. It provides the computational
backend (geometry, booleans, tessellation, I/O) while brepjs provides the
developer-facing TypeScript API.

## Architecture

Strict layered Cargo workspace. Each layer depends only on layers below it.

```
L4: brepkit-wasm        → JS bindings (wasm-bindgen)
L3: brepkit-io          → STEP, 3MF, STL, IGES, OBJ, PLY, glTF import/export
L3: brepkit-operations  → Booleans, fillets, extrusions, tessellation
L2: brepkit-algo        → GFA boolean engine, classification, intersection
L2: brepkit-blend       → Walking-based fillet and chamfer engine
L2: brepkit-check       → Classification, validation, properties, distance
L2: brepkit-heal        → Shape healing (analysis, fixing, upgrading)
L2: brepkit-offset      → Solid offset engine (global face-face intersection)
L2: brepkit-sketch      → 2D parametric constraint solver (GCS)
L1: brepkit-topology    → B-Rep data structures (arena-based)
L1: brepkit-geometry    → Curve sampling, extrema, geometry conversion
L0: brepkit-math        → Vectors, matrices, NURBS, predicates
```

### Layer dependency rules

Enforced by `scripts/check-boundaries.sh` — run before pushing:

| Crate | Allowed deps |
|-------|-------------|
| `math` | *(none — no workspace deps)* |
| `geometry` | `math` |
| `topology` | `math` |
| `algo` | `math`, `topology` |
| `blend` | `math`, `topology` |
| `heal` | `math`, `topology`, `geometry` |
| `check` | `math`, `topology`, `geometry` |
| `offset` | `math`, `topology`, `geometry` |
| `sketch` | *(none — no workspace deps)* |
| `operations` | `math`, `topology`, `algo`, `blend`, `heal`, `check`, `geometry`, `offset`, `sketch` |
| `io` | `math`, `topology`, `operations` |
| `wasm` | all crates (`blend` only transitively, via `operations`) |

The script checks `[dependencies]` in each `Cargo.toml`. A violation fails the pre-push hook.

**Allowed `use` paths per crate:**
- `math/src/**` → only `std`, external crates
- `geometry/src/**` → `brepkit_math::*`
- `topology/src/**` → `brepkit_math::*`
- `algo/src/**` → `brepkit_math::*`, `brepkit_topology::*`
- `blend/src/**` → `brepkit_math::*`, `brepkit_topology::*`
- `heal/src/**` → `brepkit_math::*`, `brepkit_topology::*`, `brepkit_geometry::*`
- `check/src/**` → `brepkit_math::*`, `brepkit_topology::*`, `brepkit_geometry::*`
- `offset/src/**` → `brepkit_math::*`, `brepkit_topology::*`, `brepkit_geometry::*`
- `sketch/src/**` → only `std`, external crates
- `operations/src/**` → `brepkit_math::*`, `brepkit_topology::*`, `brepkit_geometry::*`, `brepkit_algo::*`, `brepkit_blend::*`, `brepkit_heal::*`, `brepkit_check::*`, `brepkit_offset::*`, `brepkit_sketch::*`
- `io/src/**` → `brepkit_math::*`, `brepkit_topology::*`, `brepkit_operations::*`
- `wasm/src/**` → all `brepkit_*`

## Module Map

Quick reference — find the right file for any task:

### L0: math (`crates/math/src/`)
| Task | File(s) |
|------|---------|
| Points, vectors, matrices | `vec.rs`, `mat.rs` |
| Planes | `plane.rs` |
| NURBS curve evaluation/manipulation | `nurbs/curve.rs` |
| NURBS surface evaluation | `nurbs/surface.rs` |
| NURBS knot insertion/removal | `nurbs/knot_ops.rs` |
| Bezier decomposition | `nurbs/decompose.rs` |
| Bezier clipping intersection | `nurbs/bezier_clip.rs` |
| Curve/surface fitting (LSPIA) | `nurbs/fitting.rs`, `nurbs/surface_fitting.rs` |
| Point projection onto curves | `nurbs/projection.rs` |
| Self-intersection detection | `nurbs/self_intersection.rs` |
| 3D curves (Line, Circle, Ellipse, Parabola, Hyperbola) | `curves.rs` |
| 2D curves (Line2D, Circle2D, Ellipse2D) | `curves2d.rs` |
| Analytic surfaces (Cylinder, Cone, Sphere, Torus) | `surfaces.rs` |
| Surface-surface intersection | `nurbs/intersection.rs` |
| Analytic-analytic intersection | `analytic_intersection.rs` |
| AABB / bounding boxes | `aabb.rs` |
| BVH (bounding volume hierarchy) | `bvh.rs` |
| CDT (constrained Delaunay) | `cdt.rs` |
| Convex hull | `convex_hull.rs` |
| Filtered exact predicates | `filtered.rs` |
| Float tolerance | `tolerance.rs` |
| Geometric predicates (orient2d/3d) | `predicates.rs` |
| Ray-triangle intersection | `ray_triangle.rs` |
| 2D polygon offset | `polygon_offset.rs` |
| 2D polygon ops (clip, fillet, chamfer) | `polygon2d.rs` |
| SIMD batch operations | `simd.rs` |
| Parametric geometry traits | `traits.rs` |
| NURBS basis function evaluation | `nurbs/basis.rs` |
| Surface evaluator (power-basis cache) | `nurbs/evaluator.rs` |
| Polynomial power-basis (Horner evaluation) | `nurbs/power_basis.rs` |
| Orthonormal reference frame | `frame.rs` |
| Oriented bounding box (PCA + SAT) | `obb.rs` |
| Chord deviation arc discretization | `chord.rs` |
| Gauss-Legendre quadrature | `quadrature.rs` |

### L1: geometry (`crates/geometry/src/`)
| Task | File(s) |
|------|---------|
| Uniform curve sampling | `sampling/uniform.rs` |
| Deflection-adaptive sampling | `sampling/deflection.rs` |
| Arc-length-uniform sampling | `sampling/arc_length.rs` |
| Curvature-adaptive sampling (NURBS) | `sampling/curvature.rs` |
| Surface grid sampling | `sampling/surface.rs` |
| Point-to-curve projection | `extrema/point_curve.rs` |
| Curve-to-curve distance | `extrema/curve_curve.rs` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andymai/brepkit](https://github.com/andymai/brepkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

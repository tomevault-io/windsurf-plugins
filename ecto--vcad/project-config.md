---
trigger: always_on
description: Instructions for AI agents working on vcad.
---

# CLAUDE.md

Instructions for AI agents working on vcad.

## Overview

vcad is an open-source parametric CAD system aiming to replace Fusion 360, Onshape, and similar tools. It features a custom BRep kernel written in Rust, a React/Three.js web app, and AI-native interfaces via MCP.

**Live app:** https://vcad.io

## Commands

```bash
# Rust
cargo test --workspace             # run all tests
cargo clippy --workspace -- -D warnings  # lint — must pass clean
cargo fmt --all --check            # formatting check
cargo build --workspace            # build everything

# TypeScript
npm ci                             # install deps
npm run build --workspaces         # build all TS packages
npm test --workspaces --if-present # run tests

# App
npm run dev -w @vcad/app           # run web app locally

# Supabase (database)
supabase db push --dry-run         # preview migration changes
supabase db push                   # apply migrations to production
supabase db diff -f name           # generate migration from local changes
```

## Supabase

Cloud sync uses Supabase (Postgres + Auth). Config and migrations live in `supabase/`.

**Project:** `yteuhwciuxcbjwmabawj` (linked via `supabase link`)

**Tables:**
- `documents` — synced .vcad files (RLS: users own their docs)
- `document_versions` — automatic version history on content changes

**Adding a migration:**
1. Create `supabase/migrations/NNN_description.sql`
2. Test locally: `supabase db reset` (if running local Supabase)
3. Deploy: `supabase db push`

**Auth:** Google and GitHub OAuth configured in Supabase dashboard (not in config.toml to avoid secret leakage).

**Client:** `@vcad/auth` package wraps Supabase client. Sync logic in `packages/auth/src/sync.ts`.

## Architecture

```
vcad/
├── crates/                        # Rust workspace (~35K LOC)
│   ├── vcad-kernel-math/          # Linear algebra, transforms, exact predicates
│   ├── vcad-kernel-topo/          # Half-edge BRep topology
│   ├── vcad-kernel-geom/          # Curves and surfaces
│   ├── vcad-kernel-primitives/    # Box, cylinder, sphere, cone
│   ├── vcad-kernel-tessellate/    # BRep → triangle mesh
│   ├── vcad-kernel-booleans/      # Boolean operations (~5.4K LOC)
│   ├── vcad-kernel-nurbs/         # NURBS curves/surfaces
│   ├── vcad-kernel-fillet/        # Fillets and chamfers
│   ├── vcad-kernel-sketch/        # 2D sketch geometry
│   ├── vcad-kernel-constraints/   # Geometric constraint solver
│   ├── vcad-kernel-sweep/         # Sweep and loft operations
│   ├── vcad-kernel-shell/         # Shell and pattern ops
│   ├── vcad-kernel-step/          # STEP AP214 import/export
│   ├── vcad-kernel-drafting/      # 2D drawings, projections, GD&T
│   ├── vcad-kernel-gpu/           # wgpu compute shaders (normals, decimation)
│   ├── vcad-kernel-raytrace/      # Direct BRep ray tracing
│   ├── vcad-kernel-physics/       # Rapier3D physics simulation
│   ├── vcad-kernel-urdf/          # URDF robot description import
│   ├── vcad-kernel/               # Unified kernel API
│   ├── vcad-kernel-wasm/          # WASM bindings for browser
│   ├── vcad-ir/                   # Intermediate representation
│   ├── vcad-cli/                  # CLI tool
│   └── vcad/                      # Legacy CSG library (manifold-based)
├── packages/                      # TypeScript workspace
│   ├── app/                       # Web app (React + Three.js + Zustand)
│   ├── engine/                    # WASM engine wrapper + physics
│   ├── ir/                        # TypeScript IR types
│   ├── core/                      # Shared utilities and stores
│   ├── kernel-wasm/               # Kernel WASM package
│   ├── mcp/                       # MCP server for AI agents
│   ├── training/                  # ML training pipeline
│   └── docs/                      # Documentation site
├── supabase/                      # Database migrations and config
│   └── migrations/                # SQL migrations (pushed via `supabase db push`)
```

## Key Concepts

### BRep Kernel

The kernel uses **half-edge topology** (arena-based with `slotmap`) for boundary representation:

- **Vertex** → point in 3D
- **Edge** → curve segment between vertices
- **Face** → bounded surface region
- **Shell** → connected set of faces
- **Solid** → closed shell with volume

Surfaces: Plane, Cylinder, Cone, Sphere, Torus, NURBS

### Exact Predicates

Shewchuk's adaptive-precision predicates via `robust` crate for robust geometric decisions:
- `orient2d`, `orient3d` — orientation tests
- `incircle`, `insphere` — containment tests
- Used in boolean face classification, trimming, mesh point-in-solid

### Boolean Pipeline (4-stage)

1. **AABB Filter** — broadphase candidate detection
2. **Surface-Surface Intersection** — analytic + sampled fallback
3. **Face Classification** — ray casting + winding number
4. **Sewing** — trim, split, merge with topology repair

### Constraint Solver

Levenberg-Marquardt with adaptive damping. Constraints: Coincident, Horizontal, Vertical, Parallel, Perpendicular, Tangent, Distance, Length, Radius, Angle, Equal Length, Fixed.

### Direct BRep Ray Tracing

Pixel-perfect rendering without tessellation via `vcad-kernel-raytrace`:
- Analytic ray-surface intersection for all surface types
- WebGPU compute shader pipeline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecto/vcad](https://github.com/ecto/vcad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

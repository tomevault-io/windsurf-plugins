---
trigger: always_on
description: This file is the single source of truth for AI coding agents (Claude Code, Cursor, etc.). `CLAUDE.md` is a symlink to this file — **always edit `AGENTS.md`, never `CLAUDE.md`**. The constraints below describe the current design and the rules we work under; if a request conflicts with one of them, push back before doing it.
---

# PolyCSS — agent guide

This file is the single source of truth for AI coding agents (Claude Code, Cursor, etc.). `CLAUDE.md` is a symlink to this file — **always edit `AGENTS.md`, never `CLAUDE.md`**. The constraints below describe the current design and the rules we work under; if a request conflicts with one of them, push back before doing it.

## What this repo is

PolyCSS is a CSS-based polygon mesh rendering engine. It paints 3D meshes by emitting one DOM element per polygon, transforming it with `matrix3d`, and letting the browser composite the result. No WebGL, no canvas-per-frame. Rasterisation only happens once, into a texture atlas; everything after that is pure DOM + CSS.

Monorepo layout (pnpm workspaces):

| Package | npm name | Role |
|---|---|---|
| `packages/core` | `@layoutit/polycss-core` | Pure math: Vec3, Polygon, scene, camera, mesh ops, atlas planning. Zero browser globals (lib: ES2020 only). |
| `packages/polycss` | `@layoutit/polycss` | Vanilla renderer + custom elements (`<poly-scene>`, etc.). Owns DOM emission, CSS injection, its own copy of atlas rasterisation. Depends on `core` only. |
| `packages/react` | `@layoutit/polycss-react` | React components + hooks. Owns its own copy of atlas rasterisation. Depends on `core` only — **NOT on `polycss`.** |
| `packages/vue` | `@layoutit/polycss-vue` | Vue 3 mirror of the React package. Owns its own copy of atlas rasterisation. Depends on `core` only. |
| `packages/fonts` | `@layoutit/polycss-fonts` | Fonts + text → extruded 3D `Polygon[]`. Hand-written TrueType (`glyf`) reader + extruder (flat/round/bevel profiles) + Google Fonts loader. Framework-agnostic (returns `Polygon[]`, no React/Vue mirror needed). Depends on `core` + `earcut`. |
| `website` | `@layoutit/polycss-website` | Astro + Starlight docs site. Not published. |
| `examples/{html,vanilla,react,vue,fontcss}` | private | Per-framework Vite apps demonstrating the minimal usage for each renderer (`fontcss` demos `@layoutit/polycss-fonts`). Workspace members so they resolve to local `workspace:^` packages. Not published. |

Public API is **mirrored** across React and Vue. Adding a hook on one side without adding the matching composable on the other is not acceptable (see "Cross-package discipline" below).

### Three-like parity surface

The native PolyCSS API keeps PolyCSS world and camera conventions. For agent-friendly Three.js ports, the monorepo also exposes explicit `*/three` subpaths:

- `@layoutit/polycss-core/three` — pure Three-like math wrappers, camera conversion, lights, and transforms.
- `@layoutit/polycss/three` — the core Three-like surface plus vanilla scene helpers.
- `@layoutit/polycss-react/three` and `@layoutit/polycss-vue/three` — mirrored framework components: `PolyThreePerspectiveCamera`, `PolyThreeOrthographicCamera`, and `PolyThreeMesh`.

These subpaths intentionally use Three-compatible public names and units: `Vector3`, `Euler`, `Object3D`, `PerspectiveCamera`, `OrthographicCamera`, `DirectionalLight`, `PointLight`, `AmbientLight`, radians for object rotations, Y-up authoring coordinates, and `camera.position` + `camera.lookAt(...)` framing. They are adapters over PolyCSS, not a Three.js runtime dependency. Geometry authored in that surface is converted to native PolyCSS coordinates with `transformPolygonsToPoly`; the Y-up → Z-up axis map is `[x, -z, y]` so winding and Lambert lighting stay right-handed. The vanilla `mountPolyThreeScene` helper defaults `textureLighting` to `"baked"` because baked Lambert is the Three-parity baseline. Dynamic lighting remains available as an explicit opt-in for live CSS light changes, but it is not the exact conformance mode.

## Rendering model — the mental model

**One visible `Polygon` → one leaf DOM element.** Leaves use canonical CSS primitives where possible and move scale into `matrix3d`; clipped solids use fixed primitives because their paint geometry becomes unstable when collapsed to 1px. Atlas-backed textured polygons pack their local-2D bounding rect (`canvasW × canvasH`) into atlas pages; source-exact textured polygons may instead carry `textureImageSource` + `texturePresentation.backend="image"` and render as direct image leaves without atlas rasterisation. The HTML tag *is* the render strategy — the renderer picks one tag per polygon based on its shape and material.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LayoutitStudio/polycss](https://github.com/LayoutitStudio/polycss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

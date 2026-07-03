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

## Rendering model — the mental model

**One visible `Polygon` → one leaf DOM element.** Leaves use canonical CSS primitives where possible and move scale into `matrix3d`; clipped solids use fixed primitives because their paint geometry becomes unstable when collapsed to 1px. Atlas-backed textured polygons pack their local-2D bounding rect (`canvasW × canvasH`) into atlas pages; source-exact textured polygons may instead carry `textureImageSource` + `texturePresentation.backend="image"` and render as direct image leaves without atlas rasterisation. The HTML tag *is* the render strategy — the renderer picks one tag per polygon based on its shape and material.

Raw MagicaVoxel `.vox` sources have a narrower baked-mode fast path: `parseVox` still returns the polygon mesh for bounds, fallback rendering, and public handles, but also preserves a `PolyVoxelSource` marker. Eligible vanilla, React, and Vue meshes render visible voxel quads as `<b>` leaves inside persistent signed-face wrappers (`t`, `b`, `fl`, `br`, `fr`, `bl`), with canonical `matrix3d(...)` transforms and projected tile4 scanline order inside each mounted face. Camera-facing culling mounts/removes those face wrappers instead of removing thousands of live brush children from the mesh root. `.vox` normalization snaps to the nearest integer CSS cell size so direct voxel matrices use integer pixel coordinates without any scale wrapper; same-color shared voxel edges get a tiny matrix-space overscan to hide compositor seams without fattening exterior silhouette edges. Brush colors still receive baked Lambert shading from the scene lights. Callers may opt into lossy `.vox` palette merging and small local face-region cleanup before greedy meshing when authored palettes contain visually redundant colors; gallery and builder route this through Mesh resolution so `Lossy` may simplify palettes while `Lossless` keeps palette colors exact. Dynamic lighting, shadows, stable DOM animation, non-exact voxel geometry, and geometry replaced via `setPolygons` fall back to the polygon renderer.

Voxel-shaped meshes are the exception to "all polygons stay mounted": meshes with at most the six axis-aligned face normals, excluding helpers/auto-center-exempt meshes, automatically mount only camera-facing leaves and patch the mounted set when the camera or mesh rotation crosses a visible-normal boundary. Non-voxel meshes keep the full leaf DOM mounted; broad camera-dependent DOM culling is not worth the mutation cost.

### Tag-as-strategy table

| Tag | Strategy | When chosen | Paint mechanism | Atlas memory |
|---|---|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [layoutit/polycss](https://github.com/layoutit/polycss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->

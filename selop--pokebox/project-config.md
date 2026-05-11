---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project is a Vue 3 + TypeScript + Three.js Pokemon card viewer with GLSL shaders.

## Commands

```bash
bun dev              # Start Vite dev server with HMR
bun run build        # Type-check (vue-tsc) + production build (Vite)
bun run lint         # Run oxlint + eslint with auto-fix
bun run format       # Prettier on src/
bun test:unit        # Run all unit tests (includes shader tests)
bun test:assets      # Verify texture files match set JSON (optional — needs local assets)
bun test:shader      # Run shader compilation and validation tests
bun test:e2e         # Playwright end-to-end tests
```

**Do NOT use bare `bun test`** — it invokes Bun's built-in test runner which lacks Vite's `@/` path aliases, `vite-plugin-glsl` imports, and Playwright's test harness. Always use the specific commands above (`bun test:unit`, `bun test:shader`, `bun test:e2e`).

To run a single test file: `bunx vitest run path/to/test.ts`
To filter by test name: `bunx vitest run -t "test name pattern"`

## Architecture

Pokebox is a Vue 3 + Three.js app that creates a parallax "window into a box" effect using real-time face tracking. The webcam tracks the user's head position and adjusts an off-axis perspective camera so the screen appears as a physical window into a 3D scene containing holographic Pokémon cards.

### Rendering pipeline

1. **Face tracking** (`useFaceTracking`) polls MediaPipe for head position → writes to `store.targetEye`
2. **Scene loop** (`useThreeScene.animate`) smoothly interpolates eye position, computes off-axis projection matrix; delegates per-frame uniform updates to `ShaderUniformUpdater` and fan animation to `FanAnimator`
3. **Off-axis camera** maps real-world eye coordinates to an asymmetric frustum so the 3D scene responds to head movement
4. **Card shaders** — Each card uses one of several holo shader types, automatically selected based on card type:
   - **Illustration Rare** (`illustration-rare.frag`): Multiple vertical rainbow bands with diagonal bars + glare, matching Pokémon illustration rare holo cards
   - **Regular Holo** (`regular-holo.frag`): Diagonal rainbow gradient with rotating bar patterns + layered radial glare, matching standard holo cards
   - **Special Illustration Rare** (`special-illustration-rare.frag`): Diagonal rainbow + fine line texture + three iridescent texture layers (iri-7, iri-8, iri-9) with pointer-responsive shifts + derived-gradient sparkle on etch relief that uses dFdx/dFdy of the foil texture as pseudo surface normals so sparkle bands follow embossed contours rather than sweeping in straight lines, with iri-1/iri-2 textures for per-texel variation, matching special illustration rare cards with silvery holographic finish. See `docs/shaders/special-illustration-rare.md`.
   - **Double Rare** (`double-rare.frag`): Birthday holo with grain texture, dual dank textures, and tilt-revealed sparkles
   - **Ultra Rare** (`ultra-rare.frag`): Metallic sparkle with fully parameterized brightness/contrast/bar controls
   - **Rainbow Rare** (`rainbow-rare.frag`): Metallic sparkle spotlight + iridescent glitter from iri-7 texture, for etched SV_ULTRA double rares
   - **Tera Rainbow Rare** (`tera-rainbow-rare.frag`): Rainbow holo overlay + metallic sparkle spotlight + dual etch sparkle layers, for Tera-tagged special illustration rares
   - **Flatsilver Reverse** (`flatsilver-reverse.frag`): Inverted-mask flat silver rainbow sheen over card border/text areas (outside artwork window) with pointer-responsive spotlight, for FLAT_SILVER+REVERSE common/uncommon reverse holos
   - **Master Ball** (`master-ball.frag`): Etch foil composite on card base for RAINBOW+ETCHED masterball holo cards
   - Shared GLSL functions live in `src/shaders/common/` and are included via `#include` (resolved by `vite-plugin-glsl`):
     - `common/blend.glsl` — blend modes (overlay, screen, color-dodge, hard-light, etc.)
     - `common/filters.glsl` — adjustBrightness, adjustContrast, adjustSaturate
     - `common/rainbow.glsl` — getSunColor, sunpillarGradient (6-hue rainbow palette)
     - `common/base-adjust.glsl` — unified brightness/contrast/saturation adjustment helper
     - `common/holo-shine.glsl` — classic TCG holo shine with mask-driven rainbow overlay at configurable angles
   - All holo types use the same base uniforms and are masked by grayscale textures (`uMaskTex`, `uFoilTex`)
   - Special illustration rare, ultra rare, and rainbow rare use iridescent textures loaded from `public/img/151/iri-{7,8,9}.webp`
   - Special illustration rare additionally loads sparkle iri textures from `public/img/151/iri-{1,2}.webp` for the tilt sparkle effect (loaded via `useCardLoader.loadSparkleIriTextures()`)

### Post-processing & tone mapping

The scene always renders through an `EffectComposer` chain: `RenderPass` → `UnrealBloomPass` → `BokehPass` → `OutputPass`. The compositor runs even when DOF and bloom are disabled (passes act as passthroughs), ensuring consistent tone mapping behavior.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selop/pokebox](https://github.com/selop/pokebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

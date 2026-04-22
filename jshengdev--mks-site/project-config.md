---
trigger: always_on
description: **Every session must end by updating this file.** If you learned something new — a preference, a pattern that works, a mistake to avoid, a decision made — append it to the relevant section below. If a section doesn't exist, create one. This file is the living memory of the project. Treat it as the source of truth that compounds over time.
---

# MKS Site — CLAUDE.md

## Self-Updating Rule

**Every session must end by updating this file.** If you learned something new — a preference, a pattern that works, a mistake to avoid, a decision made — append it to the relevant section below. If a section doesn't exist, create one. This file is the living memory of the project. Treat it as the source of truth that compounds over time.

When updating:
- Add to `## Learnings Log` at the bottom with the date and what was learned
- If the learning is a durable rule, also add it to the appropriate section above
- If a previous entry is wrong or outdated, correct it in place
- Keep entries concise — one line per learning when possible

---

## What This Is

An immersive music experience for composer **Michael Kim-Sheng**. Five distinct 3D worlds, each tied to a composition and an emotional state from real listener descriptions. The music IS the router — selecting a track in MiniPlayer enters that track's world. Each world is a scroll-driven WebGL scene where the user moves a camera along a spline path through a unique landscape. A Canvas 2D entry page serves as the threshold before WebGL loads.

## Tech Stack

- **React 19** + **Vite 7** — no Next.js, no SSR, pure SPA
- **Three.js** (vanilla, NOT React Three Fiber) — all 3D rendering
- **Lenis** — smooth scroll, exposes progress (0→1) + velocity
- **pmndrs/postprocessing** — bloom, chromatic aberration, vignette, film grain, SSAO, DOF, motion blur
- **Raw GLSL shaders** — adapted from GitHub reference repos (Nitash-Biswas, al-ro, James-Smyth, Alex-DG, daniel-ilett)
- **Vanilla CSS** — no Tailwind, no CSS-in-JS
- **Web Audio API** — AnalyserNode for MiniPlayer visualizer

### Key Dependencies
```
three, lenis, postprocessing, react-router-dom
```

## Architecture

### Active Branch: `feature/environment-worlds` (from `v0.1.0-meadow-stable`)

```
src/
  App.jsx                    — Music-as-router shell: MiniPlayer + WorldContext + react-router-dom
  EnvironmentScene.jsx       — React mount point for config-driven WorldEngine
  index.css                  — Global reset + Lenis CSS
  useScrollAudio.js          — Hook: ramps audio volume from Lenis scroll progress
  useWorldTransition.js      — Portal transition hook (FBO-based GLSL dissolves)
  DevTuner.jsx + .css        — Live parameter panel (backtick to toggle, freeze/live atmosphere)

  entry/                     — Canvas 2D entry page (zero Three.js weight)
    EntryPage.jsx            — Dithered botanical flower, cursor parallax, audio context gate

  environments/              — World configs (pure data, no Three.js imports)
    golden-meadow.js         — Landing world (/) — rolling hills, golden hour, "Innocent Awakening"
    ocean-cliff.js           — /listen — cliff geometry, stylized water, dusk DOF, "Person Against Infinity"
    night-meadow.js          — /story — same terrain at night, stars, 800 fireflies, "Small But Not Alone"
    storm-field.js           — /witness — sharp terrain, rain, lightning, "Searching Through Chaos"
    ghibli-painterly.js      — /collect — cel-shading, Kuwahara, hyper-vivid, "Never Felt Brighter"
    index.js                 — Environment registry

  meadow/                    — Three.js engine (vanilla, class-based)
    WorldEngine.js           — Config-driven orchestrator (accepts environment config objects)
    MeadowEngine.js          — Backward-compat wrapper around WorldEngine
    constants.js             — Shared constants (SECTION_T_VALUES)
    ScrollEngine.js          — Lenis wrapper, exposes progress + velocity
    CameraRig.js             — CatmullRomCurve3 S-curve spline + damped lerp + terrain follow
    TierDetection.js         — Performance tier detection (1=desktop, 2=laptop, 3=mobile)
    MeadowScene.js           — Sky dome (Preetham), fog, directional + ambient light
    TerrainPlane.js          — Configurable terrain (sin/cos hills, cliff+ocean, diamond-square)
    CloudShadows.js          — Multiply-blended shadow plane with glacial UV drift
    GrassGeometry.js         — Blade mesh generator (7-segment high LOD, 1-segment low)
    GrassChunkManager.js     — Chunk pool (20x20 units), activate/dispose/fade-in, LOD swap
    FlowerInstances.js       — 6 color types x ~133 each, clearing avoidance, toon shader
    FireflySystem.js         — Configurable count (400 optimal), additive blending, vertical bob
    AtmosphereController.js  — 5-keyframe scroll-driven interpolation (sky, grass, fog, post-FX)
    CursorInteraction.js     — Mouse-to-world raycast (y=0 plane), lerped worldPos + smoothed velocity
    PostProcessingStack.js   — EffectComposer: bloom, CA, vignette, grain, SSAO, DOF, fog, color grade
    GodRayPass.js            — Screen-space radial blur (GPU Gems 3), half-res FBO
    ScoreSheetCloth.js       — Wind-driven score sheets tumbling through meadow
    ClothSolver.js           — Verlet integration cloth physics (used by ScoreSheetCloth)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jshengdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

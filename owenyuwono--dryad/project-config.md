---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A browser prototype that **procedurally generates flora** (currently focused on a single, increasingly photoreal tree) and renders it with Three.js. The thesis that drives the whole design: an organism's form is **derived from a planet's physics + a deterministic seed**, not hand-authored — and the generation logic is meant to eventually span a planet-wide ecology (grass, cactus, kelp, trees, later fauna). Everything is procedural: no authored 3D model files. The one binary asset is a CC0 HDRI for lighting.

Stack: Vite + Three.js 0.160 (WebGL2), ES modules, `vite-plugin-glsl` for `.glsl?raw` imports.

## Commands

```bash
npm run dev        # Vite dev server → http://localhost:5173  (the way to actually see the tree)
npm run build      # vite build (also the fastest "does it compile" check)
npm run preview    # serve the production build

# Tests (there is NO `test` npm script — node:test is run directly):
node --test test/*.mjs test/*.js          # full suite
node --test test/skeleton.test.mjs        # a single test file
```

Note: `node --test test/` (a bare directory) does NOT work in this Node version — always pass file globs.

## Critical working norms (read before doing anything)

- **NEVER open a browser / Playwright / chrome-devtools / a dev server to "verify" rendering or UI.** The user runs and verifies the app themselves. All visual/rendering correctness is **USER-VERIFIED** — reason about it from the code instead. (Build + Node tests are fine and expected.)
- **Determinism is load-bearing.** All generation randomness comes from `mulberry32` (`src/rng.js`). `(envelope, seed)` MUST always produce an identical organism. There is **no `Math.random` in the generation pipeline** (the only `crypto`/random use is the "randomize seed" UI button, which is not generation). When editing a generation stage, do not change the **count or order of `rng()` draws** unless you intend to (it reshuffles every downstream value).
- **The seed never sets thickness.** Radii/proportions come from physics (`solveProportions`), not from the seed. Topology is seeded; proportions are physics.
- **Adding a new gene: append its `randomGenome` draw LAST.** Insert the draw strictly AFTER the current final draw so existing seeds' other genes + canopy + foliage stay byte-identical (otherwise every downstream draw reshuffles). A generation-affecting gene (e.g. `weep`, `trunkHeight`) MUST have an IDENTITY default value that is a no-op, so existing trees and the golden-pin canopy tests stay valid. Add the new gene to EVERY hand-built genome test fixture (`genomeSchema`/`genome`/`morphospace`/`mutate`/`presets`/`roots-determinism`) or `genomeDistance`/`resolve` will `NaN`.
- **`onBeforeCompile` GLSL is NOT caught by CI.** Shader code injected via `material.onBeforeCompile` (`barkMaterial`, `leafMesh`, `windSkinGlsl`) is compiled only at runtime in the browser — `npm run build` and node tests pass even when it's broken, and the mesh then silently VANISHES in-browser. Verify every identifier against the INSTALLED three r160 source under `node_modules/three/src/renderers/shaders/` (chunk order matters — e.g. `normal` only exists from `normal_fragment_begin` onward; r160 removed the `geometry.` struct → use `geometryViewDir`). `texelFetch`/two-arg `atan` are core GLSL ES 3.00 (WebGL2). NO backticks inside `/* glsl */`...`` comment text (closes the JS string). For INSTANCED meshes, a world-space displacement must be applied POST-`instanceMatrix` (a `project_vertex` hook), never added to the pre-instance `transformed`.

## Architecture: the two pipelines

The codebase is two halves that meet at `resolve(genome, env)`:

### 1. Generation pipeline (pure ESM, no three.js, Node-testable)

The **genome IS the archetype/grammar** — a continuous gene vector (No Man's Sky-style morphospace), NOT discrete "plant types". Flow:

```
PlanetEnvelope (gravity/medium/light/sunAngle/wind/aridity/temperature; energy='photo', biochem='carbon' locked)
  + seed
  → randomGenome(env, seed)          [genome.js]   build the continuous gene vector; env "evolves"/biases it
  → deriveTraits(genome, env)        [allometry.js] continuous scaling LAWS (NOT archetypes): one "size"
                                                    driver (trunkHeight→sizeFactor) → coupled girth / leaf
                                                    size / tier count / branch order / root scale. Identity
                                                    (×1.0 / +0) at sizeFactor=1, so default-stature plants
                                                    are byte-identical. skeleton/proportions/foliage/roots
                                                    each call it for the scale they need.
  → buildSkeleton(genome, rng, jit)  [skeleton.js] recursive branch graph (trunk → branches → fine twigs)
  → solveProportions(graph, env, g)  [proportions.js] radii (pipe model), gravity droop, tip taper — ZERO rng
  → generateFoliage(graph, genome)   [foliage.js]  leaf-cluster instance set (Structure-of-Arrays)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owenyuwono/dryad](https://github.com/owenyuwono/dryad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

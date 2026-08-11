---
trigger: always_on
description: Instructions for coding agents (Codex, Devin, Claude Code, or otherwise) working in this repo. If you're drawing or animating a scene rather than working on the library itself, read the "Drawing with sketchling" section below — it's the same reference as `.claude/skills/sketchling/SKILL.md` and `.agents/skills/sketchling/SKILL.md`, kept here too since not every agent reads a skills directory.
---

# AGENTS.md

Instructions for coding agents (Codex, Devin, Claude Code, or otherwise) working in this repo. If you're drawing or animating a scene rather than working on the library itself, read the "Drawing with sketchling" section below — it's the same reference as `.claude/skills/sketchling/SKILL.md` and `.agents/skills/sketchling/SKILL.md`, kept here too since not every agent reads a skills directory.

## Working on the library

- `npm run build` compiles TypeScript (`tsc -p tsconfig.json`) — run it before any `sketchling render` call picks up source changes, since the CLI imports the built `dist/`, not `src/` directly.
- `npm test` builds then runs the unit tests (`node --test test/*.test.mjs`): deterministic core math (geometry, IK, the gait zero-slide contract, the Tier 0 linter) and the agent manifest contract. `npm run lint` runs ESLint. Both run in CI (`.github/workflows/ci.yml`) along with a render smoke test and a two-render byte-identical determinism check. Unit tests cover what's cheaply provable in Node — visual changes still need verification by rendering a scene and looking at the actual output (see "Verifying your own work" below).
- Core (`src/core/`) has no DOM dependency on purpose — it needs to run cheaply in plain Node for the Tier 0 linter and for `sketch.scene()`/`sketch.film()` construction. Don't introduce a DOM/browser dependency there; that layer belongs in `src/render/`.
- `src/render/renderer.ts` is the animation engine's spine (mount/mountFilm, the scene builder, the per-op animation dispatch); the subsystems it coordinates live beside it, one module each — `drawon.ts` (mask-based reveal), `boil.ts` (line boil), `camera.ts`/`springs.ts`/`connectors.ts`/`particles.ts` (the per-seek live systems, in that postSeek order), `mesh3d.ts`/`limb.ts` (per-tick redraw nodes), `morph.ts`, `textures.ts`, `background.ts`, with shared internal types/ids in `internal.ts` and node-lookup helpers in `scene-query.ts`. This layer is the most load-bearing code in the repo and the trickiest to get right blind; changes here deserve an actual rendered check at a genuinely mid-motion timestamp, not just a build check (see the modules' own comments for several non-obvious GSAP behaviors already found the hard way). Renders are deterministic — the same scene at the same `--at` is byte-identical — so `cmp` against a pre-change render is a real verification tool for refactors.
- `examples/` are vocabulary demos (one concept each); `examples/story/` is a narrative sequence; `examples/launch/` is launch-video-specific source, not meant as a general example. Don't mix these purposes into one file.

## Drawing with sketchling

A hand-drawn illustration and animation vocabulary, rendered through rough.js + GSAP. Like Manim: a toolbox of primitives and animations, not a pipeline. Nothing below is mandatory — pick whatever combination of tools serves what you're actually drawing. The defaults (pacing, easing, single-pen scheduling) exist because they usually look good, not because anything enforces them; override any of them whenever the scene wants something different.

### Setup

```ts
import { sketch } from "sketchling"; // or "../src/index.js" if working inside this repo, pre-publish
const scene = sketch.scene({ width: 480, height: 420, background: "#7096c6", seed: "my-scene" });
// ... build nodes, add them, animate them ...
export default scene;
```

`width`/`height` are the world. Pass `viewport: { width, height }` too when the world is bigger than one screen and a `scene.camera()` (below) pans/zooms within it; omit it and the output frame is just the whole world, same as every scene not using a camera.

`sketch.scene(...)` also takes `look` and `texture` (see "Look and texture" below) — decide the register deliberately before drawing a single primitive, not by default. Every example this repo ships that a fresh agent is likely to see first (`examples/story/lantern-maker.ts`, most of `examples/showcase/`) happens to use `look: "ink"` with a restrained, muted-palette register — that's this corpus's own bias toward one story, not a technical default or a recommendation. `"flat"` (crisp, ligne-claire/Tintin-adjacent), `"clay"` (stop-motion cadence), `"lit3d"`/`"toon3d"` (real lit 3D), and `texture: "pixel"`/`"watercolor"`/`"grain"` are equally real, equally finished registers — the same authored scene, painted differently, at zero extra authoring cost. Two different scenes about entirely different subjects that both end up ink-on-parchment isn't a coincidence this library forces; it's an unexamined default worth actively pushing against.

```
sketchling render scene.ts --out preview.png            # settled end state
sketchling render scene.ts --out mid.png --at 0.6        # a specific timeline moment
sketchling render scene.ts --crop --out thumb.png        # cropped to content (cheap iteration)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnayGarodia/sketchling](https://github.com/AnayGarodia/sketchling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

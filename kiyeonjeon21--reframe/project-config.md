---
trigger: always_on
description: Shared guide for any agent working in this repo (Claude Code reads it via
---

# reframe — instructions for coding agents

Shared guide for any agent working in this repo (Claude Code reads it via
`@AGENTS.md` in `CLAUDE.md`; Codex reads `AGENTS.md` natively). This file is the
source of truth — put durable project instructions here, not in tool-specific
files, so both tools stay in sync.

Declarative motion graphics research prototype. The loop: `scene.ts` (eDSL →
plain-JSON IR) → preview editing (recorded as non-destructive overlay JSON) →
deterministic mp4 render. Human edits survive AI regeneration of the base.

## Commands

- `pnpm reframe render <scene.ts|.html> [--overlay f] [-o out]` — mp4 into `out/`
- `pnpm reframe batch <scene.ts> <data.json|csv>` — one mp4 per row (row keys are overlay addresses like `nodes.<id>.<prop>`)
- `pnpm reframe logo <logo.svg | brand-slug> [--motion <preset>] [--energy n] [--seed n]` — animate a logo into a sting (published CLI command; `packages/render-cli/src/logoSting.ts`)
- `pnpm reframe labels <scene.ts>` — print the compiled event clock (every timeline label → exact seconds; the timing source for `audio.cues` and beat debugging)
- `pnpm reframe player <scene.ts|.json> [-o out.html]` — bundle a scene into ONE self-contained HTML that plays the motion live in any browser (and pastes into a Claude.ai Artifact). esbuild IIFE of core + `renderer-canvas` + the scene on a `<canvas>` rAF loop, with the Inter fonts inlined; visual-only (no audio / image-node sources). Entry `packages/render-cli/src/player.ts`.
- `pnpm reframe preview` / `new <name>` / `motion <mp4>` / `trace <ref.mp4>` / `guide [--regen]` / `demo`
- `pnpm test` (vitest), `pnpm typecheck`

## Authoring scenes — read the guide first

Before writing or modifying any scene (.ts), **read
`benchmark/guides/edsl-guide.md`** — it is the complete, current syntax.
A scene `.ts` file can live anywhere on disk — `render`/`batch` bundle it with
esbuild and resolve `@reframe/core` themselves, and the preview lists scenes
from the invoking directory alongside `examples/scenes/`. The repo's showcase
scenes stay in `examples/scenes/`. Scenes must be pure functions of time:
no `Math.random()`/`Date` (use `wiggle` with a seed, or pass a `seed` knob).

## Motion vocabulary (presets, path node, motionPath)

- `motionPreset(name, { target, energy, speed, intensity, from, seed })`
  (`packages/core/src/presets.ts`) returns a goal-2 `beat`. Six presets:
  draw-bloom, punch-in, rise-settle, slide-bank, reveal-orbit, spin-forge. Each
  is a **seeded generator**, not a template: same `(name,knobs,seed)` → identical
  IR; a different `seed` varies it within the same family (gated by the
  trajectory tests in `packages/core/test/presets.test.ts`).
- `path` node — vector SVG (`d`) with `progress` draw-on and `originX/Y` pivot.
  `d` is animatable: `tween(id,{d:other})` morphs the shape vertex-by-vertex
  (Lottie-style) when both `d`s share command structure; arcs `A` can't morph
  (`packages/core/src/interpolate.ts`).
- `motionPath(target, points, opts)` — Catmull-Rom curve driving x/y (+ tangent
  `autoRotate`); holds the end. Pure math in `packages/core/src/path.ts`.
- Gradients (`packages/core/src/gradient.ts`) — `fill`/`stroke` on rect/ellipse/path
  accept a `Gradient` (`Paint = string | Gradient`) via `linearGradient`/`radialGradient`/
  `conicGradient`. Coords are normalized to the node's bbox (0..1); the renderer
  (`renderer-canvas` `resolvePaint`) builds the Canvas gradient in node-local space.
  **Static** (not keyframed) — animate the NODE's transform and the gradient sweeps with
  it. Additive/golden-safe: a string fill takes the exact existing path (no new op fields);
  a gradient bypasses the string-coercing `opt()` and a path op gains a `bbox`. See
  `examples/scenes/gradient-demo.ts`.
- Shadow / glow / blur (`packages/core/src/effects.ts`) — `blur` / `shadowColor` /
  `shadowBlur` / `shadowX` / `shadowY` on drawable nodes (screen-pixel space), built with
  `glow(color,blur)` / `dropShadow(color,blur,x,y)`. **Animatable scalars** (sampled via
  `num`/`opt` → pulse a glow with `oscillate(id,"shadowBlur",…)`, pull focus with
  `tween(id,{blur:0})`). The renderer sets `ctx.filter`/`ctx.shadow*` after `setTransform`
  (per-op `save/restore` isolates them). Additive/golden-safe (absent → no op fields).
  No-op on `group` (composite blur is a later add). See `examples/scenes/shadow-demo.ts`.
- Blend modes — `blend?: BlendMode` on drawable nodes selects compositing with what's
  beneath (`screen`/`add` additive light, `multiply` tint, `overlay`/`soft-light` grade,
  …; default `normal`). **Discrete** (a static string, not keyframed); the renderer maps
  it to `ctx.globalCompositeOperation` after `setTransform` (`add`→`lighter`), isolated by
  the per-op `save/restore`. Additive/golden-safe (absent/`normal` → no op field). No-op on
  `group` (whole-subtree blend is a later add). See `examples/scenes/blend-demo.ts`.
- Track mattes — `GroupProps.matte?: MatteMode` (`"alpha"|"luma"`): a matte group's FIRST
  child masks the rest (alpha = where opaque, luma = where bright). The first feature using
  **offscreen subtree compositing**: `evaluate` emits `matte-push`/`matte-sep`/`matte-pop`
  boundary-marker DisplayOps (only for matte groups → goldens byte-identical); the renderer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kiyeonjeon21/reframe](https://github.com/kiyeonjeon21/reframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->

---
trigger: always_on
description: Popkorn is a portable **format** for motion graphics and a small **runtime**
---

# Claude Code Notes

## Vision

Popkorn is a portable **format** for motion graphics and a small **runtime**
that plays it: a hand-authored CSS-subset format, played by a zero-dependency
Canvas2D engine (with SVG and React Native/Skia backends). Its capability target
is parity with Lottie players in *rendering and animation*, not After Effects
tooling. The differentiator is that the format is hand-authorable, diffable, and
LLM-friendly, where Lottie JSON is machine-generated and opaque. Every feature
decision flows from that:

- **CSS idiom first.** When adding a capability, use the existing CSS
  property/semantics if one exists: motion paths are `offset-path`/
  `offset-distance`/`offset-rotate`, holds are `step-end`, staggering is
  negative `animation-delay`, layering is `z-index`. Never invent syntax CSS
  already has.
- **Zero runtime dependencies.** Hand-rolled parser, no build step, Canvas2D.
  A tree-sitter version existed and was deleted as overkill; don't reintroduce
  heavyweight machinery.
- **Declarative reactivity over scripting.** `var()`/`input(cursor.x)`
  bindings instead of JS expressions. If a use case demands more, extend the
  binding vocabulary (e.g. a `wiggle()` primitive), don't add a script engine.

## Architecture (and its load-bearing invariants)

Pipeline: `@popkorn/parser` `parse(source)` → typed-CSS AST (flat, knows no
shape semantics) → `@popkorn/player` `buildSceneGraph` → scene tree →
`RenderLoop` → `Renderer` interface → `Canvas2DRenderer`. The playground is a
TanStack Start app on Cloudflare Workers (usepopkorn.dev) wrapping the
`<popkorn-player>` web component.

Invariants that keep the system correct — violating any of these is how bugs
have actually happened here:

1. **`scene/transform.ts` is the single source of truth for transform math.**
   Render walk and hit-testing both consume `computeLocalMatrix`/
   `computeWorldMatrix` (motion-path placement and transform-origin included).
   Never reimplement transform composition elsewhere; there were once three
   divergent copies and the hit-boxes were wrong.
2. **Per-frame value resolution order is fixed:** reset to `node.base`
   snapshot → `var()`/`input()` bindings → animation sampling (global
   timeline) → `:hover`/`:active` overrides last. Nothing may write animated
   state outside this walk; base snapshots are immutable and deep-copied
   (gradients, shape data).
3. **`animation/registry.ts` is the only path to animatability.** A property
   animates iff it has a registry entry (number/color/gradient/path kinds).
   Geometry entries must set the relevant dirty flags (outline length, text
   bounds) — the caches (`cachedOutlineLength`, `cachedTextBounds`,
   polystar commands) trust them.
4. **Timeline is a pure function of time.** One global clock, `seek(t)` twice
   gives identical frames; per-subtree `time-offset`/`time-scale` transform
   inherited time during the walk. Never store per-animation wall-clock state.
5. **Paint order = document order among siblings, modified only by
   `z-index`** (stable sibling sort); hit-testing uses the same order
   reversed. Visibility windows (`visible-from`/`visible-until`) gate both.
6. **The renderer clears the full device-space backing buffer before the
   viewport (fit/DPR) transform is applied**; pointer input maps CSS px →
   device px → scene coords through the inverse viewport in `InputTracker`.
   Filter/mask composites are the exception: they clear, clip and blit only
   the device region `scene/bounds.ts` computes for the subtree (buffers stay
   full-size and shared, so per-composite cost tracks the *element*, not the
   viewport — this is what makes filter-heavy scenes viable off Chrome, where
   canvas filters aren't GPU-backed). `subtreeDeviceBounds` MUST stay a
   superset of what the walk paints: it mirrors the walk's `hidden`/
   `displayNone`/`isMaskSource` gating, adds each node's own filter bleed on
   the way OUT of the recursion (a blurred descendant widens its ancestor),
   and pads for stroke, antialiasing and text ink. Under-report and content
   gets clipped. A mask region is the CONTENT's box — intersect with the mask
   only when the mode is NOT inverted, since an inverted mask preserves
   content by being transparent.
7. **The three renderer backends (Canvas2D, SVG, Skia) must never hand-copy
   paint semantics from each other** — Skia drifted this way once. Rendering
   *decisions* live in the shared walk (`runtime/loop.ts renderNode`) or the
   shared helpers (`renderer/gradient-geometry.ts`, `paint-state.ts`,
   `stroke.ts`); backends keep only platform realization. Keep the `Renderer`
   interface primitive-level — don't raise it to `renderNode(node)` (SVG's
   retained diffing and Skia's per-frame RN canvas need the primitive seam).
   Any per-backend rendering fix or new backend capability gets a case in the
   cross-backend conformance suite (`renderer/conformance.ts`, one spec table
   run against all three); deliberate divergences (Skia luma·alpha limit,
   text/image no-ops, SVG text-measure approximation) are pinned there as
   expected-divergence tests — extend that table, don't silently change them.

## Lottie converter


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayarse/popkorn](https://github.com/ayarse/popkorn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->

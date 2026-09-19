---
trigger: always_on
description: Working guide for agents and maintainers. Read this before changing rendering or interaction code.
---

# AGENTS.md - Plasma UI handoff

Working guide for agents and maintainers. Read this before changing rendering or interaction code.

## What this is

`@cruxgarden/plasma-ui` - React library. All `<Plasma>` elements on a page render as **one shared plasma** on a single WebGL2 canvas behind the DOM: surfaces fuse on contact, refract the background, stretch when moved, and snap to a grid. Zero runtime deps beyond React. MIT.

- `PlasmaProvider` - owns the renderer and all shared settings; renders the canvas unless `canvas={false}`.
- `PlasmaCanvas` - the canvas element, placeable and styleable by the consumer. The renderer still covers the viewport.
- `Plasma` - marks an element as a plasma surface; polymorphic `as`; optional drag/snap/group/offset/padding.
- `usePlasmaRuntime()` (stable: `renderer`, `supported`, `reducedMotion`, `pulse`, `bump`), `usePlasmaDefaults()` (`tint`, `opacity`, `frost`, `radius`, `grid`, `magnet`, `spring`), `usePlasma()` (both).

The DOM stays ordinary HTML (text, focus, a11y). The canvas only _draws_; it never owns content.

## File map

```
src/shaders.ts        GLSL. makeShaders(maxShapes) compiles all fragment sources.
src/renderer.ts       WebGL pipeline + per-frame shape tracking. The heart.
src/spring.ts         springValue (velocity-tracking value) + animateSpring integrator.
src/snap.ts           Pure snap math. Unit-tested.
src/moods.ts          Color/spring presets, hex utils.
src/PlasmaProvider.tsx  React shell, the two contexts, PlasmaCanvas, CSS fallback, reduced
                      motion, and the shared helpers DEV / useLatest / useIsoLayoutEffect.
src/Plasma.tsx        Surface component: registration, drag, keyboard, offsets, padding.
site/                 Docs + playground, built WITH the library. site/build.mjs -> single html.
tests/                node:test suites. snap + spring are pure math; renderer.test.mjs
                      drives the GL lifecycle against tests/webgl-harness.mjs (fake
                      WebGL2 + DOM, deterministic rAF shim); ssr.test.mjs renders the
                      components through react-dom/server; tests/types/ is compile-only
                      and checked by `npm run typecheck:app`, never executed.
docs/demo.gif         README capture.
```

## Render pipeline (per frame, renderer.ts `draw`)

0. **Background** -> full-res texture `rtBg`: procedural mood field, or the `background` image (cover-fit, slow swirl + pulse warp; loaded async in `loadBackground`). If any surface has frost > 0: two blurred copies `rtBgM`/`rtBgH` (medium/heavy).
1. **Silhouette** (`maskFrag`) -> `rtA` at half res. All shapes as one SDF; smin blending.
   1b. **Tint/frost/elevation** (`tintFrag`, MRT) -> `rtT` (rgb=tint premultiplied, a=opacity) + `rtFr` (r=frost, g=elevation). Distance-weighted per-shape mix so values blend across joins.
2. **Blurs** (`blurFrag`): light blur of silhouette -> smoothed outline (traced at 0.5 contour, bicubic-sampled in comp); same light blur applied to tint and frost/elevation layers; heavier chain -> `rtC` height field.
2b. **Background blur** (`backgroundBlur` > 0): the background goes down to half res, through three ping-pong blur pairs, and back into `rtBg`, so every later pass reads the softened field for free. Eight extra passes, and none when it is 0.
3. **Composite** (`compFrag`): refraction from height-field slope with chromatic dispersion, frost = fade sharp->blurred bg copies, tint mix (opacity 1 = flat color: shimmer and bg-bleed scale by `1 - talpha`), rim (iridescent | solid | per-tint), pointer highlight, elevation-driven shadow (offset+strength from elevation channel; sampled slightly above for the caster), film grain **background only** (`grain = 1 - plasmaAlpha`).

### SDF seam rules (do not regress)

- `cornerRadii()` (renderer): a corner touching a neighbor is **squared off** (radius -> 0). The neighbor must actually overlap that corner on the perpendicular axis - both conditions in each check.
- `sdBoxG` returns a `sharp` flag for squared corners; `scene()` multiplies smin strength by `(1-sharp)` for both operands AND by direction: `k *= clamp((1 - dot(n1,n2))/.8, 0, 1)`. Net effect: flush edges stay perfectly straight, gaps/steps get fillets.
- Shapes are drawn from `elementBox()` = getBoundingClientRect with the centered pulse `scale` removed (so seams stay aligned during pulses).

## Motion systems (three, independent)

1. **Viscous surface** (renderer, per shape): each plasma box is a 4-edge spring chasing its element in **page** coordinates (scroll excluded on purpose). Drawn box = union(element, spring) so content never exits the plasma. `viscosity` maps to stiffness/zeta; `stretch` divides stiffness. Springs reset when a shape leaves the viewport.
2. **Offset springs** (Plasma.tsx + spring.ts): drag/keyboard/controlled `offset` animate `transform: translate3d`. `springValue` keeps a 6-sample ring; `getVelocity()` reads across samples because Chrome batches pointermoves per rAF (same-ms sets update the last sample in place - this fixed a real throw-velocity bug, keep the ring).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CruxGarden/plasma-ui](https://github.com/CruxGarden/plasma-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->

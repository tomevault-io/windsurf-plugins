---
trigger: always_on
description: You are working with **fluidkit**, a zero-dependency WebGL2 fluid simulation library.
---

# fluidkit — instructions for AI coding agents

You are working with **fluidkit**, a zero-dependency WebGL2 fluid simulation library.
Use it to add interactive fluid effects to landing pages, hero sections, and creative sites.
This file tells you everything you need — you should not need to read the library source.

## Setup

```sh
npm install @dilukangelo/fluidkit
```

```ts
import { createFluid, dye, threshold, custom } from '@dilukangelo/fluidkit'

const fluid = createFluid(canvas, options)
```

The canvas must have CSS size (e.g. `width: 100vw; height: 100vh; display: block`).
Add `touch-action: none` if pointer interaction is enabled. The library handles
device-pixel-ratio, resize, tab-visibility pause, off-viewport pause, context-loss
recovery, and `prefers-reduced-motion` automatically.

**SSR frameworks (Next.js, Nuxt, SvelteKit):** importing fluidkit is safe on the server,
but only call `createFluid` in browser lifecycle hooks (`useEffect`, `onMounted`).
Always call `fluid.destroy()` on unmount.

```tsx
// React: use the built-in wrapper
import { FluidCanvas } from '@dilukangelo/fluidkit/react'
<FluidCanvas render={threshold({ ... })} emitters={{ pointer: true }} className="hero-bg"
  onReady={fluid => { /* splats, fluid.params, masks */ }} />
// Options are read at mount; tune live via onReady. Or manage createFluid/destroy yourself.
```

Other adapters: `@dilukangelo/fluidkit/vue` (FluidCanvas component, `:options` + `@ready`),
`@dilukangelo/fluidkit/svelte` (`use:fluid={{ ...options, onReady }}` action),
CDN script tag `https://unpkg.com/@dilukangelo/fluidkit/dist/fluidkit.iife.js` → `window.fluidkit`.

Sound reactivity: `const a = await createAudioEmitter(fluid, { source: mediaElement })` from
`@dilukangelo/fluidkit/audio` — omit source for the microphone (prompts). `a.destroy()` to stop.

Bubbles (soda carbonation): `threshold({ ..., bubbles: { density: 0.55, rise: 1, size: 0.035 } })`.
Animated masks: pass a video (auto) or `{ live: true }` with a canvas you redraw —
`fluid.setEmitterMask(videoEl)` re-uploads every frame.

## API surface

```ts
createFluid(canvas, {
  render?: RenderMode   // dye() | threshold() | displacement() | custom(); default dye()
  emitters?: {
    pointer?: boolean | {
      color?: Color | Color[]  // fixed color or palette cycled per pointer; omit = rainbow
      intensity?: number       // dye per splat, default 0.15 (raise for threshold looks)
      radius?: number          // multiplier on splatRadius, default 1
      dragOnly?: boolean       // true = only while pressed; default false (hover emits)
    }
    ambient?: boolean | {      // autonomous wanderers, motion without interaction
      strength?: number        // ~0.2–0.5
      count?: number           // default 3
      colors?: Color[]         // palette; omit = rainbow drift
      radius?: number          // multiplier on splatRadius, default 0.6
    }
  }
  onFrame?: (t, dt) => void    // per-frame hook — drive emitters here, no own rAF needed
  respectReducedMotion?: boolean // default true — leave it on
  // ...plus any sim param below as an initial value
})

fluid.splat(x, y, dx, dy, { color, radius })  // x,y in [0,1], y UP (0 = bottom). dx,dy ≈ ±800
fluid.params.curl = 45                        // every sim param is live-tunable
fluid.setRenderMode(mode)                     // swap looks at runtime
fluid.setAmbient({ strength, colors } | null)  // reconfigure/disable ambient wanderers live
fluid.setEmitterMask(src, { color, strength })// dye pours from a text/image mask; null = off
fluid.setObstacle(src)                        // fluid flows AROUND the mask; null = off
fluid.reset()                                 // clear the fields
fluid.screenshot()                            // PNG data URL of the current look
fluid.getTexture('dye' | 'velocity' | 'pressure') // WebGLTexture for three.js/pixi
fluid.pause(); fluid.resume(); fluid.destroy()

textMask('your brand', { size: 0.35, weight: 900, font: 'system-ui', aspect: 2 })
// → white-on-transparent canvas for setEmitterMask/setObstacle
```

Sim params (defaults): `simResolution` 128, `dyeResolution` 1024, `curl` 30,
`pressureIterations` 20, `pressure` 0.8, `velocityDissipation` 0.2,
`densityDissipation` 1.0, `gravity` 0, `wind` 0 (horizontal dye drift, ± texels/s),
`speed` 1 (time scale — 0.5 = slow motion), `splatRadius` 0.25, `splatForce` 6000.

WebGPU (experimental): `const f = await createFluidGPU(canvas, opts)` from
`@dilukangelo/fluidkit/webgpu` — dye look + splats + live params only; feature-detect with
`isWebGPUSupported()` and fall back to `createFluid`. Prefer the WebGL2 core for anything styled.

## Choosing a look — the two aesthetics

**Smoke/plasma (soft, glowy):** `dye()` render, `dyeResolution` 1024, `curl` 30–50.
Good for dark hero backgrounds and cursor trails.

**Liquid/sticker (flat, posterized — the "soda" look):** `threshold()` render. The recipe
that makes it read as *liquid* instead of smoke — use all three together:
- `dyeResolution: 256` (or 128) — low res + linear filtering = smooth metaball outlines
- high cutoffs (0.3–1.6) — culls thin wisps so only dense fluid renders
- `curl: 0–5` — low vorticity flows in sheets instead of billowing

```ts
threshold({

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dilukangelosl/fluidkit](https://github.com/dilukangelosl/fluidkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

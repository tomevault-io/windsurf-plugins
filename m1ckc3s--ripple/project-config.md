---
trigger: always_on
description: Context and working notes for AI agents (and humans) continuing this project.
---

# CLAUDE.md

Context and working notes for AI agents (and humans) continuing this project.
This is an **open-source** repo — keep everything here professional and free of
private detail.

## What this is

`Ripple` is a web recreation of **Minsang's (@radiofun8)** "Ripple with Noise"
Metal shader: a distortion transition that expands from a tap point and dissolves
one photo into another (glowing, noise-warped wavefront with chromatic split).

It is **inspired by and recreated from** his work — not a literal port. The
original is Metal/MSL; this is GLSL/WebGL with meaningful changes (see
"Deviations from the original"). Credit him in any public-facing copy.

## Stack

- **React 19 + TypeScript + Vite** (standard scaffold).
- **Raw WebGL** — one fragment shader on a full-screen quad. No three.js / R3F /
  OGL.
- **GSAP** — animates a single `progress` uniform `0 → 1`; the shader does the
  rest on the GPU.

## Layout

| File | Role |
| --- | --- |
| `src/components/RippleTransition.tsx` | Everything: GLSL source (VERT/FRAG), WebGL setup, texture loading, GSAP trigger/scrub, `Params` type, `DEFAULT_PARAMS`, `EASE_OPTIONS`. |
| `src/components/Controls.tsx` / `.css` | Collapsible, shadcn-style control panel (top-left) + Dev/Scrub section. Holds the `open` collapse state, the "Controls" pill, and the close (✕) button. |
| `src/App.tsx` | Wires the component to the controls; holds `params` + `scrubValue` state. |
| `public/image-a.png`, `image-b.png` | Demo images (Pinterest placeholders — not owned; see README). |

## Running it

- Dev server: `npm run dev` → **http://localhost:3000** (pinned via
  `server.port: 3000` + `strictPort: true` in `vite.config.ts`, matching the
  owner's other repos).
- `.claude/launch.json` has a single `vite dev` entry on port 3000, started via
  the Claude preview tool. Keep it to one entry — do not add a second
  `vite preview` server.

## How the effect works (fragment shader)

1. **Wavefront** — `waveFront = progress × waveSpeed`. Distance from `u_center`
   (the normalized tap point) is compared to it. A Gaussian envelope around the
   front × a `cos(delta × waveFreq)` term defines the bright ripple band.
2. **Noise warp** — two cartesian FBM layers (`p*4` and `p*12`, value-noise +
   Hermite smoothing) perturb the distance field into cloud lobes. Amplitude is
   scaled by `warpScale = smoothstep(0.0, 0.05, progress)` so it starts as a
   small clean seed, then the `noiseWarp` slider has full authority.
3. **Displacement** — band pixels pushed radially out (`pushAmt`, melt look).
4. **Chromatic aberration** — R/G/B sampled at offset UVs (`caStrength`).
5. **Color-dodge glow** — band blown toward white (`glow`).
6. **Two-image reveal** — behind the front, `base` mixes into `target`.
   `u_swap` (0/1) flips which texture is base vs target.
7. **Tail gate** — envelope fades out by `progress` 1 so nothing lingers.

## Interaction model

- Click the canvas → ripple fires from the click point.
- **Ping-pong:** on tween complete, `state.swap` toggles and `progress` resets to
  0 *in the same frame*. The new base equals the just-revealed image, so there's
  no flicker — successive clicks alternate A→B, B→A, …
- **Click guard:** `animating` flag ignores clicks until the current transition
  finishes (no mid-animation restarts/double-swaps). `scrub()` clears it.
- **Dev scrub slider** sets `progress` directly (kills any tween) for
  frame-by-frame inspection. It scrubs the current direction.

## Controls panel & responsiveness

- **Collapse pattern** (matches the sibling `shimmering-dots` repo, but anchored
  **top-left** instead of bottom-right): the panel and a "Controls" pill share
  one fixed `.rc-root` anchor and cross-fade via scale + opacity, toggled by the
  `open` state. The ✕ in the header collapses; the pill re-opens. Header buttons
  (Reset + ✕) are matched to 26px; the bottom **Replay** uses the outlined
  uppercase `.rc-secondary` style.
- **`.rc-root` is `pointer-events: none`** — it's only a positioning anchor and
  is sized to the (sometimes hidden) panel, so leaving it interactive made it
  swallow taps over the canvas even while collapsed. The pill and the open panel
  re-enable `pointer-events: auto` themselves.
- **Breakpoint width:** mobile (`≤640px`) caps the panel at the **same 280px** as
  desktop, not wider — otherwise the panel *grew* when crossing into mobile.
  Below ~312px it shrinks via `calc(100vw - 32px)`.
- **Image sizing** is computed once at mount from `window.innerWidth`: desktop
  uses 0.7w / 0.86h, mobile (`≤640px`) uses 0.9w / 0.8h. It adapts on load /
  rotation-then-reload, **not** live on desktop window drags (would require
  re-running the WebGL setup on resize).
- **Touch lock:** `html, body, #root` are `overflow: hidden` +
  `overscroll-behavior: none`, and the image wrapper is `touch-action: none`, so
  a touch-drag fires a tap (ripple) instead of scrolling/panning the canvas.

## Deviations from the original Metal shader

- **Removed the scatter/dissolve block.** In the original it flung pixels behind
  the wave to random UVs and faded to black. It caused a "shake," a hard edge,
  and a destructive image-scramble. Replaced with a clean two-image reveal.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m1ckc3s/ripple](https://github.com/m1ckc3s/ripple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

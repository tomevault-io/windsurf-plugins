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
| `src/components/RippleTransition.tsx` | The effect: GLSL source (VERT/FRAG), WebGL setup, texture loading, GSAP trigger/scrub. Default-exports only the component. |
| `src/components/rippleParams.ts` | Non-component module: the `Params` / `RippleHandle` types, `DEFAULT_PARAMS`, `EASE_OPTIONS`. Split out of `RippleTransition.tsx` so that file only exports a component (satisfies `react-refresh/only-export-components` / Fast Refresh). |
| `src/components/Controls.tsx` / `.css` | Collapsible, shadcn-style control panel (top-left) + Dev/Scrub section. Holds the `open` collapse state, the "Controls" pill, and the close (✕) button. |
| `src/App.tsx` | Wires the component to the controls; holds `params` + `scrubValue` state. |
| `public/image-a.png`, `image-b.png` | Demo images (Pinterest placeholders — not owned; see README). |

## How the effect works (fragment shader)

1. **Wavefront** — `waveFront = progress × coverage`, where
   `coverage = 1.0 + 0.5*noiseWarp + 0.1` is auto-derived so the front always
   reaches the farthest corner (normalized distance maxes at 1.0) plus the noise
   margin by `progress` 1 — the sweep completes on any canvas/aspect. Distance
   from `u_center` (the normalized tap point) is compared to it. A Gaussian
   envelope around the front × a `cos(delta × waveFreq)` term defines the bright
   ripple band. (There is no Wave Speed uniform — see "Wave Speed → Transition
   Speed" below.)
2. **Noise warp** — two cartesian FBM layers (`p*4` and `p*12`, value-noise +
   Hermite smoothing) perturb the distance field into cloud lobes. Amplitude is
   scaled by `warpScale = smoothstep(0.0, 0.05, progress)` so it starts as a
   small clean seed, then the `noiseWarp` slider has full authority.
3. **Displacement** — band pixels pushed radially out (`pushAmt`, melt look).
4. **Chromatic aberration** — R/G/B sampled at offset UVs (`caStrength`).
5. **Color-dodge glow** — band blown toward white (`glow`).
6. **Two-image reveal** — behind the front, `base` mixes into `target`.
   `u_swap` (0/1) flips which texture is base vs target. The reveal boundary is
   feathered by `feather = 0.04 + 0.05·noiseLarge` so it reads as an organic edge,
   not a hard ring.
7. **Tail gate** — envelope fades out by `progress` 1 so nothing lingers.

## Interaction model

- **Press the canvas → ripple + pinch fire from the press point.** The whole
  effect is bound to `pointerdown` (mouse/touch/pen), not click — there is no
  release/click path. `pointerdown` is guarded to the primary button, and pairs
  with the image wrapper's `touch-action: none` so a press can't be a scroll.
- **Pinch poke:** a snappy push-in dimple fires together with the wave when the
  `pinch` toggle is on. Its depth is scaled by `pinchStrength` — the pinch tween
  peaks at `pinchStrength` (no separate uniform; `u_pinch` already multiplies the
  displacement). On by default at strength 0.3.
  - **Geometry (shader).** The dimple is a Gaussian `pinchG = exp(-dist²/2σ²)`
    with `pinchSigma = 0.10`. The radial displacement is its *slope*
    (`pinchDisp = (dist/σ²)·pinchG·0.01·u_pinch`), so the pull is zero at the
    exact contact point and far away, maxing around the rim — the sheet reads as
    a physical lens dent that bends the picture, not painted-on shading. The
    `0.01` factor is the hand-tuned displacement scale.
  - **Sign convention.** `uvOffset = dir·(pushAmt − pinchDisp)` — *subtracting*
    the pinch makes the band sample outward, so content gets sucked toward the
    tap (the "pushed-in" look).
  - **Frame pin / edge-fade.** `edgeFade = smoothstep(0, 0.14, dist-to-nearest-
    border)` multiplies the dimple to zero as it nears any edge. Without it the
    dent could drag the sample out of bounds, where `CLAMP_TO_EDGE` smears the
    border and bleeds the other image in. Like paper anchored in a frame, the
    very edge can't deform.
  - **Contact shadow.** A soft `color.rgb *= 1 − 0.16·pinchG·edgeFade·u_pinch`
    pools shade in the bottom of the dimple for depth. Pure Gaussian, no
    high-frequency detail, so it never adds hard radiating lines; the `0.16`
    depth is kept subtle so the geometric distortion stays the star.
- **Ping-pong:** on tween complete, `state.swap` toggles and `progress` resets to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m1ckc3s/ripple](https://github.com/m1ckc3s/ripple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->

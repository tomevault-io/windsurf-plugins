---
trigger: always_on
description: Turn a raster logo (PNG/JPG/WebP/screenshot) into a clean minimal SVG with edge smoothness as the primary hard gate and IoU optimized as high as reasonably possible without a fixed global threshold, then into a choreographed logo animation delivered as standalone JS-rendered HTML, applying Disney's 12 animation principles. Use when asked to animate a logo, build a logo reveal / splash screen / brand intro, convert a logo image into animated SVG or HTML, add motion to a vectorized mark, or create
---


# Pixel2Motion (v2)

**Pixel → Vector → Motion.** This skill fuses two disciplines:

1. **Vectorization discipline** (from logo-vectorizer-minimal-smooth): fit the raster source with the *lowest-complexity* editable geometry that passes visual QA.
2. **Motion discipline** (from Disney's 12 Principles of Animation): choreograph that geometry into brand-appropriate motion with evidence-based timing and easing.

The fusion rests on one insight: **minimal smooth geometry IS animatable geometry**. A logo built from 3 semantic parts animates cleanly; a logo built from 400 pixel-stair traced points cannot be choreographed or accepted as a professional logo vector. Phase 2 therefore does not merely vectorize — it *structures for motion*. And the animation must always land exactly on the QA-verified static vector (the **Final Frame Contract**).

## Deliverables

- `logo.svg` — final static vector (motion-ready structure)
- `logo_motion.html` — showcase-style standalone HTML with the main animation, atomic motion studies, replay/slow/speed controls, and QA hooks
- `motion_spec.md` — personality words, principles applied, timeline table, easing tokens, atomic motions, tunable controls
- `outputs/fit_iterations/*.png` + `overlay_progress_strip.png` — geometry QA evidence
- `outputs/motion_frames/*.png` + `motion_strip.png` — motion QA evidence
- `final_render.png`, `html_render.png` — static renders; path audit artifacts when smoothness was a concern

---

## Phase 1 — PIXEL: Read the source, write the brief

1. **Analyze the raster source.** Record image size, mode, alpha, foreground colors, background (transparent/solid). Identify the *semantic parts*: mark vs wordmark, individual letters, dots, swooshes, containers, negative space. Each part is a future actor in the animation.
2. **Write the motion brief** before touching geometry:
   - **Personality**: 3 brand motion words (e.g. "swift, precise, confident"). Derive from the logo's own visual language (geometric = engineered motion, organic = flowing motion, rounded = soft/bouncy) plus any user-stated brand context. See `references/motion-personality.md` to map words → timing scale, easing tokens, principle emphasis.
   - **Usage context**: splash/intro (deliberate 1200–2000ms), header reveal (300–800ms), loading state (continuous loop), hover micro-interaction (150–300ms). Ask the user only if genuinely ambiguous; default to a splash-style reveal plus a static end state.
   - **Choreography sketch**: which parts move, in what order, using which reveal pattern (see `references/reveal-patterns.md`).
3. Record the brief in `motion_spec.md`. Every later parameter must trace back to it.

## Phase 2 — VECTOR: Fit minimal geometry, structured for motion

Follow the lowest-complexity-first workflow. **Do not maximize pixel-fit by default**; start with the simplest editable geometry that can explain the mark, escalating only when an overlay shows a structural mismatch.

### Complexity Ladder

Use the first level that matches the source well enough:

1. **Primitives**: circles, ellipses, rects, lines, simple arcs, transforms.
2. **Primitive composites**: boolean-like combinations of a few primitives or masks.
3. **Few-curve analytic paths**: a small number of cubic segments for smooth ribbons, swooshes, C-marks, leaves, waves, shields. For closed and/or self-intersecting variable-width ribbons (∞ marks, scripts) use the centerline-scaffold recipe — `references/ribbon-fitting.md` + `scripts/fit_ribbon_centerline.py`; its report also emits each crossing's arc fractions, which Phase 3's split draw-on needs.
4. **Smoothed outline paths**: more knots only where the source has real shape changes; preserve G1 tangent continuity, no noisy handle flips.
5. **Trace-derived paths**: only for irregular silhouettes where simpler geometry fails (`scripts/raster_logo_trace.py` for measurement/starter masks). Trace output is a measurement aid, never automatically final art. Smooth or refit the trace before delivery.

If a higher-complexity version improves only pixel antialiasing, keep the lower one. If a lower-complexity version has wrong endpoints, width profile, center, silhouette, negative space, or visibly stair-stepped edges, move up one level or refit with smooth curves. Prefer live SVG `<text>` for wordmarks unless exact letterforms are required. Decide the provisional complexity from the source itself and record the reason in `motion_spec.md`.

### Smoothness Gate (hard requirement)

IoU is not allowed to hide bad vector craft. **A smooth logo source must ship as smooth vector geometry.** The final SVG fails geometry acceptance if any intended smooth edge visibly stair-steps, chatters, has pixel-grid orthogonal runs, contains noisy trace knots, or looks like a bitmap mask at 200-400% zoom, even when IoU is numerically high.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nolangz/pixel2motion](https://github.com/nolangz/pixel2motion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

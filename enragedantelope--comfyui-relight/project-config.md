---
trigger: always_on
description: A single, self-contained ComfyUI node that adds up to 3 positionable light sources to any image — colored additive light, precise color correction, or both — with presets, directional gradients, rim lighting, and mask-aware subject occlusion including a cast shadow. Fast and deterministic: pure image processing (numpy + Pillow + scipy), no diffusion pass, no models to download. Built on the ComfyUI v3 node schema, with a small frontend in `web/`.
---

# AGENTS.md — comfyui-relight

A single, self-contained ComfyUI node that adds up to 3 positionable light sources to any image — colored additive light, precise color correction, or both — with presets, directional gradients, rim lighting, and mask-aware subject occlusion including a cast shadow. Fast and deterministic: pure image processing (numpy + Pillow + scipy), no diffusion pass, no models to download. Built on the ComfyUI v3 node schema, with a small frontend in `web/`.

## Current state

_Last verified: 2026-09-04_

- **Status:** v4.0.0 is on branch `v4-overhaul`, not yet merged or released; `main` is still v3.1.2. Published to the Comfy Registry via `.github/workflows/publish_action.yml`, which fires on a `pyproject.toml` version change on **`main`** — so the bump is safe on a branch, and a functional change on main needs a bump or it never ships.
- **Works:** up to three independent light sources; three lighting modes (colored additive, two-zone grading, and both in that order); radial-falloff and directional-gradient mask shapes; subject interaction in front of / behind the subject, the latter with a rim highlight, a falloff background glow and a traced cast shadow; the built-in preset set; a debug view that follows the `debug_image` wiring; conditional widget visibility and a working node recreate. `.github/workflows/test.yml` runs pytest across Python 3.10–3.12, a `node --test` frontend job, and a separate ruff job.
- **In progress:** v4.0.0 on `v4-overhaul`, now **exercised in a live ComfyUI** (0.34.0 / frontend 1.51.9) via Playwright: node registration, conditional visibility and its round-trip, preset greying, the legacy-workflow migration against the real v3.1.2 save, single-run debug wiring, and "Fix node (recreate)" all behave, with no ReLight console warnings. Browser QA found two defects, both fixed here — see the v4.0.0 changelog. The three retuned presets have been rendered and signed off; "Rim Light (Behind)" carried a `(200, 255, 200)` light colour from v1.0 that put a +20/255 green cast on the rim, and is now neutral white at `light_intensity` 1.0.
- **Known gaps / next steps:** output quality depends heavily on the input mask, and the only mask-quality guard is a console-only warning when a mask is >90% white; presets are plain dicts at the top of `relight.py` with no way for a user to add their own without editing the file; there is no example beyond the bundled workflow JSON; the rim highlight still does per-frame CPU SciPy work (`fg_mask[b].cpu().numpy()` + Sobel per frame) because vectorising it risks numeric drift, though the cast shadow added in v4.0.0 is vectorised in torch.
- **Deep docs:** none — `README.md` is the user-facing reference and `relight.py` is the whole implementation.

## Architecture in 60 seconds

- **Single node.** `relight.py` contains the entire feature — one node class, all logic self-contained.
- **Up to 3 independent light sources.** Each with position, mode (colored light or color correction), mask shape (circular falloff or gradient), and fine-tuning controls.
- **Three lighting modes per source:** colored additive RGB light, precise color correction (brightness, contrast, saturation, temperature, tint, gamma), or both — colour first, then the grade applied to the lit result.
- **Mask shapes:** radial falloff (inner/outer radius) or directional gradient (sunset rays, window light).
- **3D subject interaction** (needs a mask): light in front of the subject, or behind it — rim highlight, a background glow with real falloff, and a cast shadow traced by `cast_shadow_mask` (a `grid_sample` ray march, run at `_SHADOW_TRACE_MAX` and upsampled; no SciPy).
- **Built-in presets.** Soft Window Light, Dramatic Side Light, Warm Sunset Glow, Cool Blue Moonlight, Studio Key Light, Rim Light (Behind), Spotlight, Negative Light (Darken).
- **Visual debugging with no toggle.** Wiring the `debug_image` output is the whole gesture; see the convention below for why that needs a hidden input.
- **A frontend that hides what is inert.** `web/` carries the legacy-workflow migration, the debug-output tracking, conditional widget visibility and a working "Fix node (recreate)"; `tests/frontend/` drives all four outside a browser.

## Layout

| File | Purpose |
|------|---------|
| `__init__.py` | ComfyUI custom-node entry point (registers the ReLight node) |
| `relight.py` | The entire node: lighting engine, presets, UI widgets, image processing |
| `web/relight_migrate.js` | Remaps a pre-v4 save's positional widget values by name on `onConfigure` |
| `web/relight_debug.js` | Keeps `debug_output_connected` in step with the `debug_image` wiring, and hides it |
| `web/relight_ui.js` | Conditional visibility: hides irrelevant blocks, greys what a preset overrides, fits the node |
| `web/relight_recreate.js` | A "Fix node (recreate)" that replaces instead of duplicating |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnragedAntelope/comfyui-relight](https://github.com/EnragedAntelope/comfyui-relight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

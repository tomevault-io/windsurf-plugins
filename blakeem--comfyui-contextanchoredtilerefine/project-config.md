---
trigger: always_on
description: ComfyUI custom node package, three nodes over ONE tile geometry and TWO engines: the base
---

# Context-Anchored Tile Refine, project guide

ComfyUI custom node package, three nodes over ONE tile geometry and TWO engines: the base
node's raster path (`sampling._refine_tiles`) and the VL nodes' synchronized path (`sync.py`
over `stepper.py`), which since 1.6.0 is the only VL path. They refine an
already-upscaled IMAGE by dynamic tiling (or only a masked region, leaving the rest
untouched; upscaling happens outside the node), except the all-in-one variant which
upscales in-node first. Target: ComfyUI 0.3.45+, V1 node schema, Python 3.12, torch 2.9.

A MiniMax H3 VIDEO node (`ContextAnchoredTileUpscaleVLVideo`, `video.py`, `vl_video.py`)
was built and then REMOVED on 2026-08-13: the spatial tile method is seamless on static
shots and unusable under camera motion, where the seam is a fixed line in a moving field
that no anchor/overlap setting removes. What was learned is kept, not the code —
`docs/h3-video-chunking-findings.md` holds the full result set and the temporal-chunking
approach that replaces it, and `tests-AB/` keeps every H3 harness. Do not re-add it
without that doc's temporal design.

## Prime directives (highest priority, override convenience)

1. **Quality first, efficiency second.** Output image quality is the top priority and is never
   traded for speed, memory, or simpler code. Optimize only *after* quality is guaranteed, and
   never in a way that risks a visible quality regression. **Never resize, resample, or apply any
   lossy operation to a tile.** Tiles are extracted, processed, and pasted back at their native
   pixel size (multiples of 8 by construction).

2. **Never re-diffuse finished pixels that survive into the output.** Running diffusion again over
   already-refined content compounds grit with the samplers this node targets, visibly, even once
   and even untiled. This is the root constraint behind every seam decision below.

3. **Seams are hidden by conditioning, not by blending.**
   - *Between tiles, BASE node (the raster path, `sampling._refine_tiles`):* each tile is sampled
     oversized (core + `context_overlap` + a frozen
     `context_anchor` halo). The anchor halo encodes from the live canvas, so the tile sees its
     already-refined neighbors and is drawn to continue them. On sides bordering an
     already-processed neighbor (top/left in raster order), the `context_overlap` band is diffused
     from the FROZEN RAW source by both tiles independently, and the two results are cross-dissolved
     by a thin directional feather. Prohibited: a wide blend of two independent refinements
     (ghosting), and a double hard-paste of a shared strip (compounding artifacts).
   - *Between tiles, VL nodes (the sync path, `sync.py`):* the seam is PREVENTED, not hidden —
     there is no earlier tile to hide it from. Every tile is a lane of ONE run over ONE shared
     canvas latent, all stepped together per sigma, and the per-step consolidation feathers the
     bands at LATENT scale and scatters the result back, so no two lanes ever disagree on a shared
     cell at a step start. Both sides of every band therefore decode ONE latent, which is why this
     path runs NO min-error cut and NO DC match: neither has anything left to correct. The raster
     rules above still govern the base node, unchanged.
   - *At a mask boundary:* no feather at all. The masked region is diffused against the frozen
     background as context, then composited back with a 1px anti-alias only. An inward feather
     would under-process a ring around the subject; an outward feather would re-diffuse finished
     background (directive 2).

## Architecture (respect these invariants)

- `context_anchored_tile_refine/node.py`: the V1 nodes (`INPUT_TYPES` / `VALIDATE_INPUTS` /
  `refine`). `ContextAnchoredTileRefine` normalizes and validates the optional MASK;
  `ContextAnchoredTileRefineVL` subclasses it (required CLIP, no prompt input) and
  routes through `refine_image(vl_clip=...)`; `ContextAnchoredTileUpscaleVL` is the
  all-in-one variant (widgets replace the NOISE/SAMPLER/SIGMAS/GUIDER inputs, optional
  UPSCALE_MODEL + negative, no mask) — it runs `upscale.prepare_upscaled` on the whole
  image, builds the sampling objects via `upscale.py`, and calls the same
  `refine_image(vl_clip=..., sampler_name=...)` — the widget NAME rides along beside the built
  SAMPLER so a sampler the sync engine rejects is named as the user picked it (core's
  `sampler_object` wraps several names in a private function). Both VL nodes carry two selects,
  defined ONCE each as `_anchor_source()` / `_vlm_method()` so their option lists and tooltips
  cannot drift apart, and both APPENDED after `context_overlap` (see the ANCHOR RING invariant
  for why never mid-list). `anchor_source` takes its option strings from `sync.ANCHOR_SOURCES`
  and `vlm_method` from `captions.vlm_methods()`, so what the widget offers and what the engine
  branches on cannot diverge. Comfy-free at module scope (the combo lists come from a lazy
  `import comfy.samplers` inside `INPUT_TYPES`, and the option strings from lazy package
  imports).
- `context_anchored_tile_refine/grid.py`: pure grid math (tile layout: `core`,
  `overlap_inner_rect`, `crop_rect`, `paste_rect`; `solve_axis`, `build_layout`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blakeem/ComfyUI-ContextAnchoredTileRefine](https://github.com/Blakeem/ComfyUI-ContextAnchoredTileRefine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

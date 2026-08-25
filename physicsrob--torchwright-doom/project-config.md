---
trigger: always_on
description: This submodule is the active DOOM renderer: a computation graph that
---

# Current state of this submodule

This submodule is the active DOOM renderer: a computation graph that
`torchwright` compiles into a transformer which renders DOOM
autoregressively. `torchwright_doom/model/render_main.py` is the entry
point — `forward(input_vec, past)` builds the read-side scene and
protocol views, publishes the runtime protocol owners, builds each
dispatch branch's next-token, and selects one by the current input
token's type. Canonical numbers (resolution, token counts, timings):
`FACTS.md`.

Layout of `torchwright_doom/`. The top-level split is the dumb-host
line: **`model/`** holds every module that compiles into transformer
weights; everything else runs as Python on the host.

- **`model/` root — the machine kernel** (flat files = shared
  substrate): `vocab` / `tokens` / `value_ranges` / `marker_ranges`
  (the token vocabulary and value encodings), `embedding` / `emit` /
  `extract` (token ↔ residual encode/decode), `std` (the helper-op
  shim), `past` / `attention_handles` (reading previously-emitted
  tokens), `token_match` (token-type match predicates), `render_ops`
  (shared math: atan2, distance, clamps), `constants` (env-driven
  screen sizing) / `render_constants` (attention gains + protocol
  sentinels), `doom_lighting` / `asset_config` (the data floor under
  the token contract — see `GLOSSARY.md`), and `render_main` (the
  assembler — the front door).
- **`model/scene/`** — the static read side: prefill token
  interpretation, header recovery, queryable map facts, the assembled
  SceneIndex.
- **`model/protocol/`** — the autoregressive protocol: current-token
  interpretation and the declarative dispatch/ownership table.
- **`model/traversal/`** — the BSP walk: `bsp_traversal` /
  `bbox_pruning` / `traversal_edges` (R_RenderBSPNode + visibility
  pruning) and `solid_intervals` (the solidsegs occlusion channel the
  walk queries — original DOOM keeps solidsegs in `r_bsp.c` too).
- **`model/raster/`** — seg projection through pixels, one
  deliberately chunky stage: `seg_*` (wall-segment projection),
  `wall_range_*` / `wall_column_*` (wall columns), `visplane_*` /
  `flat_*` (floor/ceiling planes), `pixel_dispatcher` / `uv_compute`
  (final pixels), `psprite_renderer` / `statusbar_renderer`, and the
  dispatch glue `range_dispatcher` / `payload_router`. The
  segs/walls/planes/pixels sub-split is deliberately not directories —
  those clusters import each other bidirectionally; the filename
  prefixes do the grouping.
- **`model/assets/`** — `wad_assets` / `asset_banks` / `assets` /
  `hud_assets` / `weapon_assets` (textures, flats, HUD and weapon
  graphics compiled to lookup tables), `lighting` (colormaps), and
  `pwl_banks` (texture-coordinate PWL wraps for the pixel pass, *not*
  lighting — distinct from `emit.py`'s digit-quad PWL).

`model/__init__.py` carries the per-module map and the import-layering
contract (kernel → {protocol, assets} → scene → traversal → raster;
`render_main` is the only kernel module that imports the stage
directories), enforced by `tests/architecture/test_runtime_policy.py`.

- **Lifecycle packages** (everything below the graph, one per stage) —
  `tokenizer/` (the token contract: per-id labels, the stock WordLevel
  tokenizer, frozen vocab data, row arithmetic in `rows.py`, the raw-word
  codec in `codec.py`), `prompt/` (input side: WAD → scene subset → prompt
  rows; production entry `prompt/scene.py`), root `infer.py` (the sole
  portable inference program, copied byte-identical to each bundle's root
  and executed as a subprocess — never imported), `interpret/` (output
  side: emitted rows → pixels, PNGs, metrics against the pydoom oracle,
  the prettifier wrapper), `bundle/` (publication: `build.py` compile +
  staged rollback publication, `manifest.py` schema/validation,
  `layout.py` copied files), `portable/` (pure-stdlib tool sources shipped
  under bundle `tools/`), `diagnostics/` (ONNX diagnostics, never on the
  production path), and the root job spec `config.py` / `identity.py` with
  the orchestrator `run.py` and the dispatch-only `cli.py`.

**Reading path** for one `forward()` pass, read side → write side
(all under `model/`): `vocab` / `tokens` → `embedding` / `extract` →
`scene/` (static read side) → `protocol/` (the dispatch table) →
`render_main.forward` (assembly) → the write side:
`traversal/bsp_traversal` (R_RenderBSPNode) → `raster/seg_projection`
→ the `wall_*` / `visplane_*` / `flat_*` rasterizers → the pixel pass. The **prefill pipeline** (WAD →
the tokens the model reads before autoregression) is `prompt/wad.py` →
`prompt/subset.py` → `prompt/build.py` → `prompt/scene.py` →
`tokenizer/rows.py` (row indices); `README.md` spells out both chains
with file:function references.

Each renderer module is ported from an original plain-Python counterpart;
many docstrings note that provenance.

See `GLOSSARY.md` for the coined vocabulary (carrier, head, lifted key,
radix, digit-quad, flat, visplane, …).

# Dumb host principle

The DOOM renderer is an exercise in extreme constraints. The goal is
not to render DOOM efficiently — it's to render DOOM in a way that is
entirely analogous to autoregressive LLM inference. Each step has a
discrete input token and predicts a discrete output token. The host

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [physicsrob/torchwright_doom](https://github.com/physicsrob/torchwright_doom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->

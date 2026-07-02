---
trigger: always_on
description: Last updated: 2026-06-30
---

# CLAUDE.md — krea2-explorations

Last updated: 2026-06-30

Project memory for this repo. Global conventions (uv, TDD, path-privacy, docs, no emojis) are in the
user-level CLAUDE.md and still apply; this file holds only what's specific to this repo.

## What this is

Tools for measuring and surgically editing Krea 2's text conditioning (per-layer projector edits,
checkpoint analysis, attention extraction) plus the experiment harnesses that test them. Measure-first:
lead with falsifications, not "discoveries".

## Two lever classes (keep both in mind)

Two ways to steer Krea 2's conditioning, both in scope here:

1. **Weight/activation edits** — the projector rebalance + single-layer isolation tooling (the core package:
   `projector`, `projector_lora`, `comfy_nodes`), plus the **concept-direction** nodes
   (`krea2_concept_inject_node`): *Krea 2 Concept Direction* measures a difference-of-means axis from an A/B
   prompt pair in-graph, *Krea 2 Concept Inject* amplifies/injects/project-outs it on the conditioning
   (`scripts/concept_direction.py` is the offline CLI that writes the same `.npy`). Edits weights or
   activations, stays in-distribution via downstream RMSNorm. Guide: `docs/concept_directions.md`;
   workflow: `example_workflows/krea2_concept_inject.json`.
2. **Prompt-side steering** — a `<think>` block / system prompt / prefix written into the text the encoder
   sees. Inject custom spans via the **tokenizer skip-template route**: pass a full `<|im_start|>…` string as
   the prompt and the qwen3vl tokenizer emits it verbatim, so no ComfyUI/pipeline edit is needed. It behaves
   like a steering vector — push within-distribution and prompt adherence holds. See `docs/findings.md`
   ("Prompt-side steering").

Public/tracked files stay benign in name and content; sensitive prompts, data, and any
sensitive-referencing filenames live only in gitignored `internal/` and `data/`.

## Public-facing docs — keep in sync (these are the front door)

When you add or change a **public tool** (a ComfyUI node, a `scripts/` CLI, or a package capability), update
the user-facing docs in the SAME change — they're what people actually use:
- `README.md`: the "With the toolkit you can" bullets, a short TL;DR section, and the **Components** table.
- `docs/`: the relevant guide (e.g. `docs/concept_directions.md`) — add one if none fits; cross-link
  `docs/findings.md`.
- Bump the `Last updated:` date on every doc you touch.

Keep all public examples benign (generic prompts/axes — expression, style, pose); anything sensitive stays in
gitignored `internal/`. Don't let README/docs drift behind the code.

## Comparison grids / figures — use the shared util

Experiment validators produce comparison contact sheets (rows = prompts/variants, cols = arms/methods).
There is ONE tested implementation — **do not re-inline PIL grid code**:

```python
from krea2_explorations.image_grid import build_contact_sheet
build_contact_sheet(grid_rows, out_path, col_labels=[...], row_labels=[...])
```

`grid_rows` is a 2D list (rows x cols) of image path / `PIL.Image` / `None`; missing cells render as a
placeholder instead of crashing. It depends only on Pillow + stdlib. Tests: `tests/test_image_grid.py`.

## Building generation graphs — reuse the built workflows, don't re-inline

NEVER hand-inline a graph dict, and NEVER copy an older harness's `graph()` as a template — a re-inlined copy
silently drifts from the *measured* recipe (this has bitten us: soft renders, a stale `ModelSamplingFlux`, the
wrong scheduler). Two already-built sources, both no-`ModelSamplingFlux`:

- **The A/B/C/D recipes are THE source of truth**, built in `scripts/canonical_workflows.py` (now TRACKED):
  `A` (drop-in), `B` (cfg-headroom), `C` (the two-sampler Turbo-LoRA split), `D` (SDE finish), plus
  `build_single` / `build_split`. They encode the canonical decision — the **modular `SamplerCustomAdvanced`
  stack, `beta57` scheduler, bf16 qwen3vl encoder, fp8 RAW DiT, krea2RealVae** (filenames single-sourced from
  `generate.py`). Call A/B/C/D for ANY canonical render (internal or public; `use_res=False` for a fixed 1024²
  latent — `Krea2Resolution` needs a ComfyUI restart to load).
- **`scripts/generate.py` is the low-level layer beneath** the recipes: `build_graph` (single-`KSampler`
  primitive), `run`, `resolve_vae`, `model_node`, the model-filename constants — and `build_split_graph`, a
  low-level two-stage split primitive (two `KSamplerAdvanced`, `scheduler=simple`) for harnesses that wire their
  own conditioning, NOT the canonical workflow C. Documented in `docs/krea2_inference.md`.
- **A custom node composes onto a build_graph seam, it doesn't re-inline.** A harness adding its own node hangs
  it on one of two seams instead of copying the skeleton: `model_patches=[model_node("Krea2AttnBias", ...)]` for
  a model-edge lever (attention bias, residual steer, DiT capture), or `cond_patches=[cond_node("Krea2ConceptInject", ...)]`
  for a positive-conditioning lever (concept inject / project-out). Both keep the canonical skeleton and only add
  your node; the negative is left untouched.

Traps a re-inline (or copying an older harness's `graph()`) gets wrong:
- **No `ModelSamplingFlux`** — the 1.15 flow-shift is in Krea2's model config, so the node is a pixel-identical

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fblissjr/krea-explorations](https://github.com/fblissjr/krea-explorations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->

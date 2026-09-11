---
trigger: always_on
description: This is the canonical coding-agent guide for the repository. `AGENTS.md` is a
---

# RenderFormer Studio repository guide

This is the canonical coding-agent guide for the repository. `AGENTS.md` is a
compatibility symlink to this file so the instructions have one source of
truth.

## Working rules

- Parallelize independent work. Use processes rather than threads for
  CPU-bound batches; threads remain appropriate for streaming subprocess I/O.
- Drive batches from explicit JSON/JSONL/database metadata rather than
  discovering work with filesystem traversal. Record IDs, paths, and status in
  the metadata.
- Size worker pools for the workload and machine. Print progress with
  `flush=True` and include a worker prefix such as `[W0]` when outputs can
  interleave.
- Fail fast. Add exception handling only when there is a defined, actionable
  recovery path.
- Support Python 3.10 and 3.11. Use the active environment for CPU work; use
  `environments/cuda.yml` plus the pinned pip and FlashAttention receipts for
  CUDA, and `environments/datagen.yml` plus PyPI `bpy==4.5.10` for data
  generation. Never hardcode a machine-specific environment prefix.

## Purpose and public surface

RenderFormer Studio contains the data, model, training, inference, and
evaluation code for two model generations:

- V1/RF1: the released diffuse/specular renderer, including the public Base
  and Large checkpoints.
- V2/RF2: heterogeneous primitives, environment lighting, volumes,
  displacement, SVBRDF material latents, and mixed-resolution training.

There is one installable Python package, `renderformer`, under
`src/renderformer/`. The distribution is named `renderformer-studio`, and it
installs one executable with three subcommands:

```bash
renderformer infer --help
renderformer train --help
renderformer data --help
```

The equivalent module form is `python -m renderformer <command> ...`. It works
after an editable install, or with `PYTHONPATH=src` in a source-only
environment. Do not reintroduce repository-root inference or training
wrappers.

## Repository map

```text
src/renderformer/
  pipelines/    RenderFormerPipeline and RenderFormerV2Pipeline
  modeling.py   reusable pretrained/save adapter for RenderFormerModel
  models/       shared numerical model implementation
  data/         V1/V2 generation, H5 schemas, postprocess, and loaders
  training/     independent RF1/RF2/material runners and shared utilities
  ops/          maintained CUDA/Triton operator adapters
  cli/          unified dispatcher and inference command
configs/
  model/v1/     V1 architecture YAML
  model/v2/     V2 architecture YAML
  model/material/  material-autoencoder architecture YAML
  data/v1/      stage-numbered V1 generation YAML
  data/v2/      stage-numbered V2 generation YAML
  data/material/  material-sphere EXR generation YAML
scripts/data/   portable generation recipes and data utilities
scripts/train/  portable curriculum-stage launchers
examples/       runnable RF1/RF2 scene sources and asset provenance
docs/           environment, inference, and training guides
```

Renderer training and inference use the same `RenderFormerModel`. The
inference pipelines assemble version-specific preprocessing and auxiliary
encoders around that component. RF1, RF2, and the material autoencoder retain
separate training loops because their data and optimization flows differ;
`renderformer train` is their shared dispatcher, not a combined numerical
loop.

## Data boundaries

All maintained data code lives under `renderformer.data`:

- `renderformer.data.exporters.v1` owns the strict RF1 13-channel encoding.
- `renderformer.data.exporters.v2` owns the raw V2 export.
- `renderformer.data.textures` owns learned material postprocessing.
- `renderformer.data.h5` owns atomic H5 writing and schema validation.
- `renderformer.data.loaders.v1` owns RF1 PyTorch/DALI loading.
- `renderformer.data.loaders.multires` owns weighted RF2 multiresolution,
  environment, and volume loading.
- `renderformer.data.geometry` owns the reusable local remesh and UV helpers.
- `renderformer.data.material_exr` owns the strict manifest-driven EXR dataset,
  and `renderformer.data.material_preprocessing` owns its invertible
  `log10_1p` transform.
- `renderformer.data.material_spheres` owns deterministic planning, explicit
  render shards, verified resume, and finalization of the three-family EXR
  corpus. Rendering runs only in the active PyPI `bpy==4.5.10` Python.
- `renderformer.models.material` owns the historical material autoencoder and
  three BRDF-to-9-D-latent mapper architectures. The two V2 transformers and
  four material components share the `RenderFormer/renderformer-v2`
  repository; its subfolder layout is listed in the root README. Default mapper
  repository/subfolder pairs live with the Blender material definitions. One
  repository revision selects all six components atomically, and the default
  revision is used unless the caller explicitly requests one. The autoencoder
  release contract is alpha-premultiplied, finite, nonnegative linear RGB
  transformed by `log10(1 + RGB)` and inverted by `10**network_RGB - 1`.

Keep `import renderformer.data` lightweight. Blender, DALI, Diffusers, and
other heavy optional backends must remain behind explicit or lazy imports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamNCJ/RenderFormer-Studio](https://github.com/iamNCJ/RenderFormer-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->

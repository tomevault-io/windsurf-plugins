---
trigger: always_on
description: transforms; put new loaders and transforms in `experiments/<exp>/data.py`.
---

# Marigold V2 maintainer notes

Guidance for people and coding agents working on this repository. The README
is the user-facing document; this file explains how the code fits together and
which conventions to keep.

## Purpose and scope

Public code for "Marigold V2: Revisiting Diffusion Transformers for Monocular
Depth Estimation" (ACM TOG 2026). Three deliverables, in this priority:

1. Reproducible inference with the released checkpoints (`scripts/infer.py`).
2. Reproducible evaluation of the paper numbers (`evaluation/`).
3. Reproducible training and a framework that extends to new dense-prediction
   tasks (`marigoldv2/`).

Base model: Qwen-Image-Edit-2509 (DiT quantized to 4 bit, rank-128 LoRA,
single fixed timestep). Checkpoints: Hugging Face `huawei-bayerlab/marigold-v2-0`.
Prior work is "Marigold V1" (depth) and "Marigold V1.1" (all modalities);
their evaluation code and dataset preprocessing are reused, see NOTICE.

## Layout

```
scripts/infer.py                         inference on an image folder; builds a config and runs evaluate_pipeline
scripts/download_assets.py               all Hugging Face downloads into assets/
scripts/hypersim_{normals,albedo}/       dataset builders that run the Marigold V1.1 preprocessors scene by scene
marigoldv2/core/                         registry + builder (compose registered classes from config)
marigoldv2/dataset/                      GenericDataset, manifest loaders, per-sample transforms, collate
marigoldv2/loss/loss.py                  latent MSE, masked L1, gradient L1, SinkLoss (WindowMatchedL1Loss), iREPA
marigoldv2/validation/                   validation steps (inference, metrics, visualization, .npy writers), metrics
marigoldv2/trainer/generic_trainer.py    training loop (accelerate), validation, checkpointing
marigoldv2/script/train/                 train.py entry point, argument parsing, checkpoint save/load hooks
marigoldv2/experiments/<exp>/            per model family: network_graph.py, component_loader.py, task modules, training_*.yaml
marigoldv2/config/datasets/              dataset YAMLs referenced by training configs through base_config
evaluation/config/inference_depth.yaml   the model definition used for inference
evaluation/depth/                        five zero-shot depth benchmarks (inference -> subsets -> eval.py -> summary)
evaluation/depth_see/                    Soft Edge Error on Hypersim at native resolution
evaluation/normals/, normals_saee/       normals benchmarks and Hypersim soft angular edge error
evaluation/albedo/                       Hypersim albedo PSNR/SSIM/LPIPS
evaluation/src/                          Marigold V1 benchmark datasets, alignment, metrics (derived code, keep headers)
evaluation/data_split/                   split files used by the benchmarks and dataset builders
assets/                                  downloads (git-ignored) plus tracked examples/ and images/
```

## Environment and checks

- `bash setup/setup_env.sh [env] [cuXXX]` creates a conda env, installs the
  torch pin from that script, then `pip install -e .`. Dependencies are pinned
  in `pyproject.toml`; when changing a pin, rebuild the env and rerun the
  smoke test below. `pip check` must stay clean.
- Assets live under `assets/` or `$DEPTH_ASSETS_DIR`:
  `checkpoints/Qwen-Image-Edit-2509/` (only `transformer/` and `vae/` are
  loaded), `checkpoints/Marigold-V2/{depth/<variant>,normals,albedo,qwen_text_embeddings}/`,
  `checkpoints/dinov3-vitb16-pretrain-lvd1689m/` (training only),
  `datasets/{marigold_train,vkitti2,marigold_depth_eval,marigold_normals_eval,marigold_train_normals,marigold_train_albedo,LayeredDepth-Syn}`.
- Install the pre-commit hook once per clone, before the first commit:
  `pip install pre-commit && pre-commit install`. It runs the same two ruff
  commands CI runs, so a commit that goes through cannot fail the ruff
  workflow. `.pre-commit-config.yaml` and `.github/workflows/ruff.yml` pin the
  same ruff version on purpose: formatting rules change between releases, and
  an unpinned CI once reformatted files an older local ruff had accepted. Bump
  both together. Run it over the whole tree with `pre-commit run --all-files`.
- CPU checks that need no assets: import every module from outside the repo,
  run `--help` on every script under `scripts/` and `evaluation/`, and
  `ruff check . && ruff format --check .` (CI runs the same ruff commands).
- GPU smoke test: `python scripts/infer.py --image_dir assets/examples --output_dir output/smoke`
  for `depth` and `--modality normals`. Expect one `.npy` and one PNG per
  image at the input resolution, finite values, depth in [-1, 1].
- Inference needs about 17 GB of GPU memory at 1024² and 29 GB at 2048²;
  loading quantizes the DiT and takes a few minutes.

## How the pieces fit

- Everything is composed from YAML through the registry in `marigoldv2/core`.
  Categories: `dataset`, `manifest`, `manifest_transform`, `dataset_transform`,
  `network_components`, `network_graph`, `loss`, `validation_steps`. A config
  names classes by their registered name and passes their kwargs.
- Config loading: `recursive_load_config` merges `base_config` entries
  (repo-relative paths) and registers the OmegaConf resolvers `mul` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huawei-bayerlab/marigold-v2](https://github.com/huawei-bayerlab/marigold-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

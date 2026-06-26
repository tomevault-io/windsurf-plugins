---
trigger: always_on
description: - This file guides coding agents working in the repository root.
---

# AGENTS.md

## Purpose
- This file guides coding agents working in the repository root.
- Use it together with `README.md`.
- Prefer the maintained end-to-end workflow over older helper scripts.

## Documentation Policy
- Keep documentation minimal, practical, and reproducibility-first.
- `README.md` should stay a short hub, not a long tutorial.
- The core linked docs are `documents/data_setup.md`, `documents/pipeline.md`, `documents/hr_lr_coordinate_conventions.md`, and `notebooks/misc/hr_lr_mapping.ipynb`.
- `documents/utils_functions.md` is a developer maintenance audit for shared `src/utils`; do not treat it as a pipeline execution guide.
- `documents/data_setup.md` is the canonical data setup reference for `scripts/setup_data.py`, data sources, folder layout, and download flags.
- `documents/pipeline.md` is the canonical reproducibility and inference reference after data setup is complete.
- The documented data layout uses `<DATA_ROOT>/raw`, `<DATA_ROOT>/input`, `<DATA_ROOT>/output`, and `<DATA_ROOT>/models` as the canonical roots.
- Do not add extra tutorial, explanation, reference, or experiment-summary docs unless the user explicitly asks for them.
- Keep unresolved follow-up work out of user-facing docs unless the user explicitly asks for it there.
- Keep `README.md`, `documents/data_setup.md`, `documents/pipeline.md`, and `scripts/setup_data.py` aligned when changing data setup or workflow commands.
- Do not imply that the full workflow is completely reproducible until classification GT artifacts and code/config path alignment are settled.

## Public Branch Sync
- Do not merge `density_refactoring` into the public branch with history; copy selected file contents and create a single public-branch commit instead.
- For non-media files, update only Markdown files that already exist on the public branch.
- Licensing files such as `LICENSE` and `THIRD_PARTY_NOTICES.md` may be added to the public branch when updating repository licensing.
- Do not add private-only, old-notebook, deep-research, experiment-tracking, or other source-branch-only files to the public branch.
- Replace the public `media/` directory with the source branch `media/` directory when refreshing public media assets.
- Keep the public branch cleaned and reproducibility-focused; do not restore stale notebooks, extra research files, or non-public artifacts.

## Licensing
- Original CALHippo source code is released under Apache License 2.0.
- Keep the Apache-2.0 `LICENSE` file standard; do not add custom restrictions there.
- Model weights, trained checkpoints, datasets, derived annotations, rendered figures, notebook outputs, and other BigBrain-derived artifacts are released under CC BY-NC-SA 4.0 for non-commercial academic research use only.
- Third-party model code and dependencies remain subject to their original upstream licenses and notices; keep relevant notices in source folders and/or `THIRD_PARTY_NOTICES.md`.
- UNI2-h weights are not redistributed by this repository and must be obtained from the upstream provider.

## Repository Summary
- Language: Python.
- Python: `>=3.10, <=3.13`.
- Package manager: `uv`.
- Project name: CALHippo Framework, the framework for the Cellular Annotation Library for the Hippocampus dataset.
- Main code: `src/`.
- Main configs: `experiments/`.
- Main maintained preprocessing package: `src/preprocessing/`.
- Main maintained density package: `src/density_estimator/`.
- Main maintained LR inference package: `src/lr_inference/`.
- Main maintained path: raw BigBrain preprocessing -> HR WSI segmentation -> cell classification -> HR to LR density dataset creation -> density training -> full-slice LR inference -> 3D point cloud reconstruction.

## Rule Files
- Cursor rules in `.cursor/rules/`: none found.
- `.cursorrules`: not present.
- Copilot instructions in `.github/copilot-instructions.md`: not present.

## Environment
- Install dependencies with `uv sync`.
- Prefer `uv run <command>` in automation.
- Activate the venv interactively with `source .venv/bin/activate` if needed.
- Create and populate data through `scripts/setup_data.py`; prefer `documents/data_setup.md` for setup details and supported flags.
- Create the data tree with `uv run python scripts/setup_data.py --data-root data`.
- Download aligned HR data with `uv run python scripts/setup_data.py --data-root data --download-hr`; when no IDs are provided it uses `scripts/default_lr_ids.txt`.
- Download model artifacts with `uv run python scripts/setup_data.py --data-root data --download-weights`; private repos require `hf auth login` or `HF_TOKEN`.
- Canonical maintained region names are `RCA1`, `RCA2`, `RCA3`, and `RCA4`.
- `uv` cache is configured under `./data/uv_cache`.
- Linux GPU setups assume CUDA 12.6 for pinned PyTorch wheels.
- Segmentation mixes TensorFlow and PyTorch in the same process; VRAM pressure matters.

## Primary Workflow
- Stage 0: preprocess raw BigBrain HR/LR slices and hippocampal surfaces with `src/preprocessing/` when starting from raw inputs.
- Stage 1: segment nuclei on HR WSI crops with `src/segmentation/multimodel_inference.py`.
- Stage 2: classify segmented HR cells with `src/classification/main_classification.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AImageLab-zip/CALHippo-Framework](https://github.com/AImageLab-zip/CALHippo-Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->

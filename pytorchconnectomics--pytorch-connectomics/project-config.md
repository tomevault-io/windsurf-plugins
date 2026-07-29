---
trigger: always_on
description: Handles data loading with MONAI transforms:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PyTorch Connectomics (PyTC) is a modern deep learning framework for automatic and semi-automatic semantic and instance segmentation in connectomics - reconstructing neural connections from electron microscopy (EM) images. The framework integrates PyTorch Lightning for orchestration and MONAI for medical imaging tools, maintained by Harvard's Visual Computing Group.

## Agent Quick Reference

Map of common user intents → authoritative source files. Use this
table first; jump straight to the listed paths instead of grepping.
This is the single source of truth for agent navigation; the prompt
files under `prompts/` (`prompts/INSTALL.md`, `prompts/ADD_DATASET.md`,
`prompts/ADD_ARCH.md`, `prompts/DEBUG_TUTORIAL.md`) are thin wrappers
that point back at it.

| Intent | Authoritative source | Concrete example |
|---|---|---|
| Run training | `scripts/main.py` → `connectomics/runtime/dispatch.py` | `just train mito_lucchi++` |
| Run inference + decode + evaluate | `--mode test` → `inference/stage.py` → `decoding/stage.py` → `evaluation/stage.py` | `just test mito_lucchi++ <ckpt>` |
| Tune decode params (Optuna) | `runtime/tune_runner.py` → `decoding/tuning/optuna_tuner.py` | `python scripts/main.py --config <yaml> --mode tune --checkpoint <ckpt>` |
| Add a dataset / new EM volume | `tutorials/<new>.yaml` (copy closest); data dicts in `data/datasets/data_dicts.py`; new file format only if needed → `connectomics/data/io/io.py` | `tutorials/mito_lucchi++.yaml` |
| Add a model architecture | `connectomics/models/architectures/`; register via `@register_architecture("name")` decorator; add config params to `connectomics/config/schema/model.py` | `models/architectures/monai_models.py` |
| Add a loss function | `connectomics/models/losses/losses.py`; register in `create_loss()`; metadata in `losses/metadata.py` | `models/losses/build.py` |
| Add a decoder | `connectomics/decoding/decoders/`; register via the `register_decoder(name, fn, *, overwrite=False)` *function call* in `decoding/registry.py` (NOT a `@register_decoder` decorator) | `decoding/decoders/segmentation.py` |
| Change augmentation | `connectomics/data/augmentation/build.py`; profile YAMLs in `config/profiles/augmentation_*.yaml` | `data/augmentation/transforms.py` |
| Change postprocess | `connectomics/decoding/postprocess.py`; templates in `config/templates/decoding_*.yaml` | `decoding/streamed_chunked.py` |
| Add a tutorial config | `tutorials/<name>.yaml`; validate with `python scripts/validate_tutorial_configs.py --glob 'tutorials/<name>.yaml'` (note: `--glob` is additive over the default `tutorials/*.yaml`; filter output for the new path before fixing anything) | `tutorials/mito_lucchi++.yaml` |
| Debug a failing tutorial | `prompts/DEBUG_TUTORIAL.md`; reproduce with `python scripts/main.py --config <yaml> --fast-dev-run` | `python scripts/main.py --config <yaml> --fast-dev-run` |

When a new intent class shows up, add a row here rather than scattering
pointers across READMEs.

## Architecture Philosophy

The codebase follows a clean separation of concerns:
- **PyTorch Lightning**: Orchestration layer (training loop, distributed training, mixed precision, callbacks, logging)
- **MONAI**: Domain toolkit (medical image models, transforms, losses, metrics)
- **Hydra/OmegaConf**: Modern configuration management (type-safe, composable configs)

**Key Principle:** Lightning is the outer shell, MONAI is the inner toolbox. No reimplementation of training loops or domain-specific tools.

### V2/V3 Architecture Contract

The codebase enforces an explicit contract from the v2/v3 refactor:

1. **One canonical owner per concept.** No backward-compatibility shims, no facade re-exports, no duplicate import paths.
2. **Strict config.** Unknown top-level keys **raise** at load time. Removed fields raise. `getattr(cfg.x, "y", default)` ghost reads on undeclared fields are forbidden.
3. **Stages are separate.** Pipeline = `train → infer → decode → evaluate → tune`. Each stage has its own package and its own entry function. Combined test-mode is a thin wrapper that calls stage APIs in sequence.
4. **Dependency direction:** `config → utils → data → models → metrics`; `training → {config, data, models, metrics}`; `inference → {config, data, models}`; `decoding → {config, data, utils}`; `evaluation → {config, data, metrics}`; `runtime → {config, training, inference, decoding, evaluation}`. Static AST tests in `tests/unit/test_v3_guardrails.py` enforce this.
5. **Public API is explicit and small.** `tests/unit/test_public_api_snapshot.py` asserts exact `__all__` membership.

## Agent Design Principles

- **Ecosystem-first, no reinvention**: Leverage proven frameworks (PyTorch, Lightning, MONAI, nnU-Net) to keep the codebase modern, minimal, and scalable.
- **Config-first reproducibility**: Use Hydra/OmegaConf YAML composition + CLI overrides so experiments are declarative, reproducible, and easy to customize across datasets/benchmarks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PytorchConnectomics/pytorch_connectomics](https://github.com/PytorchConnectomics/pytorch_connectomics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

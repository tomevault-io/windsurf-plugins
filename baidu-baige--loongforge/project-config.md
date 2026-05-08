---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LoongForge is large-scale transformer training framework built on top of Megatron-LM (as a patched fork: Loong-Megatron) and TransformerEngine. It supports LLMs, VLMs (Vision-Language Models), VLAs (Vision-Language-Action Models), and Diffusion Models across both NVIDIA GPUs and Kunlun XPUs. Training phases supported: pretrain and SFT (supervised fine-tuning).

## Build & Setup

### Quick Start (Docker — recommended)
```bash
git clone --recurse-submodules https://github.com/baidu-baige/LoongForge.git
# COMPILE_ENV: ampere | hopper | blackwell
docker build --build-arg COMPILE_ENV=hopper --build-arg ENABLE_LEROBOT=false \
  -t loongforge:latest -f ./LoongForge/docker/Dockerfile .
```

### Source Install
```bash
# 1. Clone with Megatron submodule
git clone --recurse-submodules https://github.com/baidu-baige/LoongForge.git
cd LoongForge

# 2. Install LoongForge + dependencies
uv pip install -e ".[gpu]"    # NVIDIA GPU
uv pip install -e ".[xpu]"    # Kunlun XPU

# 3. Setup TransformerEngine (clone, patch, build)
python setup_env.py --te-tag v2.9
```

Note: `setup_env.py` only handles TransformerEngine. Megatron-LM (Loong-Megatron) is a git submodule at `third_party/Loong-Megatron`, initialized via `--recurse-submodules`.

### Build Package
```bash
sh build.sh    # Creates output tarball
```

## Running Tests

E2E tests use a custom YAML-driven framework (`tests/main.py`), not pytest.

```bash
# Download test datasets first
bash tests/download_datasets.sh

# Run default CI test suite (all models in tests/configs/)
bash tests/main_start.sh

# Run optional regression tests (tests/optional_configs/)
bash tests/main_start.sh --optional
```

### Running a Single Model Test

Edit variables in `tests/main_start.sh`:
```bash
# Run one model from tests/configs/
model_names="qwen3_14b"

# Run one model from tests/optional_configs/
model_names="deepseek_v2/deepseek_v2_lite"
include_optional=true

# Run an entire model series from optional_configs/
model_names="NONE"
optional_subdir="internvl2.5"
include_optional=true
```

Test configs: `tests/configs/` (CI suite) and `tests/optional_configs/` (regression, organized by model family). Each YAML defines model params and multi-step `scenarios` (checkpoint conversion + training).

## Training Launch Pattern

Training scripts use `torchrun` for distributed execution. The PYTHONPATH must include both Megatron-LM and LoongForge:

```bash
PYTHONPATH=$MEGATRON_PATH:$LOONGFORGE_PATH:$PYTHONPATH \
    torchrun --nproc_per_node 8 --nnodes $NNODES ... \
    $LOONGFORGE_PATH/loongforge/train.py \
    --model-name <model-name> \
    --training-phase pretrain|sft \
    ...
```

Key arguments: `--model-name` (maps to config via `config_map.py`) or `--config-file` (direct YAML path), `--training-phase` (pretrain/sft).

## Architecture

### Core Package: `loongforge/`

- **`train.py`** — Entry point. Calls `parse_train_args()` then `build_model_trainer(args).train()`.
- **`train/parser.py`** — Argument parsing: merges Megatron CLI args with Hydra YAML configs (OmegaConf). Supports `--model-name` (looked up in `config_map.py`) or `--config-file`.
- **`train/trainer_builder.py`** — Registry-based trainer dispatch. `register_model_trainer(model_family, training_phase)` decorator registers training functions per model family and phase.
- **`train/megatron_trainer.py`** — `MegatronTrainer` wraps model_provider, dataset_provider, and forward_step into Megatron's `pretrain()` loop.
- **`train/training_utils.py`** — Extended Megatron pretrain loop (heavily customized).
- **`train/arguments.py`** — LoongForge-specific extra CLI arguments added on top of Megatron's.
- **`train/validators.py`** — Validation logic for Megatron and LoongForge args.
- **`train/pretrain/`** — Pretrain implementations for LLM and VLM.
- **`train/sft/`** — SFT implementations for LLM, VLM, InternVL, ERNIE.
- **`train/custom/`** — Custom model trainers (e.g., WAN diffusion, Pi0.5 VLA).

### Model System: `loongforge/models/`

- **`factory.py`** — Model registry. `register_model_config(family, arch)` registers model configs; `register_model_provider(family)` registers model provider functions (accepts a single family string or list of families). Lookups: `get_model_config()`, `get_model_provider()`, `get_model_family()`.
- **`dispatch.py`** — Hardware-abstraction layer (`MultiAccModules`). Provides unified access to TransformerEngine or local linear/attention/norm implementations.
- **`foundation/`** — LLM backbone implementations: LLaMA, Qwen (all versions through Qwen3-Next), DeepSeek, InternLM, MiniMax, MIMO, GLM. Each defines a transformer spec and config dataclass.
- **`encoder/`** — Vision encoder implementations: base ViT, Qwen2-VL/3-VL, InternVL, LLaVA-OV, ERNIE-VL.
- **`omni_models/`** — Multi-modal model composition: `OmniCombinationModel` assembles encoder + projector + decoder into a unified pipeline, with `model_chunk_schedule_plan.py` for pipeline parallelism scheduling.
- **`common/`** — Shared layers (local norms, projectors, etc.).
- **`custom/`** — Non-standard models (WAN diffusion, Pi0.5).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baidu-baige/LoongForge](https://github.com/baidu-baige/LoongForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

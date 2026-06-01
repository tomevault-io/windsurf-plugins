---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ProtoMotions3 is a GPU-accelerated simulation and RL framework for training physically simulated digital humans and humanoid robots. It supports multiple physics simulators (IsaacGym, IsaacLab, Newton, Genesis, MuJoCo) and RL algorithms (PPO, AMP, ASE, MaskedMimic). Written in Python 3.8+, Apache-2.0 licensed.

## Common Commands

### Setup
```bash
pip install -e .
pip install -r requirements_isaacgym.txt  # or requirements_isaaclab.txt, requirements_newton.txt, requirements_genesis.txt, requirements_mujoco.txt
```

**MuJoCo CPU Backend**: For CPU-only testing, use `requirements_mujoco.txt`:
```bash
# Install PyTorch CPU version (lighter, no CUDA needed)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
pip install -e .
pip install -r requirements_mujoco.txt
```

Note: MuJoCo backend is CPU-only and supports `num_envs=1` only.

### Training
```bash
python protomotions/train_agent.py \
    --robot-name g1 \
    --simulator isaacgym \
    --experiment-path examples/experiments/mimic/mlp.py \
    --experiment-name my_experiment \
    --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
    --num-envs 4096 \
    --batch-size 16384

# With config overrides (these become PERMANENT in resolved_configs.pt)
python protomotions/train_agent.py ... --overrides agent.config.learning_rate=0.0001 env.max_episode_length=1000
```

### Inference
```bash
# G1 pretrained model
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/last.ckpt \
    --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
    --simulator isaacgym --num-envs 16

# SOMA pretrained model
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/soma-bones/last.ckpt \
    --motion-file data/motion_for_trackers/soma23_bones_seed_mini.pt \
    --simulator isaacgym --num-envs 16

# Sim2sim: train in isaacgym, test in newton
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/last.ckpt \
    --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
    --simulator newton --num-envs 16

# CPU-only inference with MuJoCo (single env)
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/last.ckpt \
    --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
    --simulator mujoco --num-envs 1
```

### Testing
```bash
pytest protomotions/tests/
pytest protomotions/tests/test_newton_simulator_fk.py  # single test file
```

### Linting and Formatting
```bash
# IMPORTANT: Do NOT use `pre-commit run --all-files` — many repo files don't conform yet,
# causing 100+ unrelated modifications. Instead, target specific files:
pre-commit run --files <file1> <file2> ...   # explicit file list
pre-commit run                                # runs only on staged files
pre-commit run ruff --files <file1> ...       # lint only, specific files
```

### ONNX Export (BeyondMimic Trackers)
```bash
# Reference script for BM tracker configs (auto-detects actor obs keys from checkpoint)
python deployment/export_bm_tracker_onnx.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/last.ckpt

# For non-BM configs, copy and adapt this script to match your observation keys
```

## Key Dependencies

torch, lightning (Fabric), tensordict, wandb

**Newton simulator**: Tested against Newton 1.0.0 from PyPI. Install via `pip install "newton[examples]"` — see `requirements_newton.txt`.

**MuJoCo simulator**: CPU-only backend using the `mujoco` Python package (>=3.0). Supports single environment (`num_envs=1`) for lightweight testing and debugging. Useful for quick policy validation without GPU.

## Important Files

- `examples/experiments/format.py` — documents experiment config function signatures
- `protomotions/train_agent.py` — main entry point, documents config system in detail
- `protomotions/envs/mdp_component.py` — MdpComponent design docs
- `protomotions/envs/context_views.py` — FieldPath/context system
- `protomotions/utils/simulator_imports.py` — simulator import order handling

## Architecture

### Agent Hierarchy

```
BaseAgent (abstract) — training loop, checkpoints, Lightning Fabric
├── PPO — actor-critic, GAE advantages, clipped surrogate
│   ├── AMP — adds discriminator, replay buffer, style rewards
│   │   └── ASE — adds MI encoder, latent skills, diversity loss
│   └── Mimic/ADD — adds pose tracking diff (extends AMP)
└── MaskedMimic — expert distillation (behavioral cloning, not RL)
```

All models are `TensorDictModuleBase` subclasses. Forward passes read from and write to a shared `TensorDict`. Models use `nn.LazyLinear` extensively — input shapes are inferred on first forward pass.

Key methods each algorithm implements: `create_model()`, `perform_optimization_step()`, `record_rollout_step()`, `register_algorithm_experience_buffer_keys()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVlabs/ProtoMotions](https://github.com/NVlabs/ProtoMotions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

---
trigger: always_on
description: **vla-rlt** implements Physical Intelligence's RL Token (RLT) approach on open-source VLAs for online learning from experience. The architecture is **VLA-agnostic** — swap the backend by changing one config line.
---

# vla-rlt

## Project Overview

**vla-rlt** implements Physical Intelligence's RL Token (RLT) approach on open-source VLAs for online learning from experience. The architecture is **VLA-agnostic** — swap the backend by changing one config line.

**Default VLA**: SmolVLA (450M, HuggingFace / LeRobot)  
**Target hardware**: SO-101 robot arm + M2 MacBook Air 8GB (MPS)  
**Paper**: [Physical Intelligence RLT](https://www.pi.website/research/rlt)

### Why SmolVLA?
- 450M params — fits comfortably in 8GB unified memory
- MPS-native: runs on Apple Silicon without CUDA
- Trained on SO-100/SO-101 data (exact target hardware)
- LeRobot ecosystem: data, robot drivers, all in one place

---

## Architecture

```
Observation (images + language)
        │
   [SmolVLA VLM]  ← frozen (SmolVLM-2, 960 hidden, 32 layers)
        │
   hidden states from last N layers
        │
   [RLT Encoder] ──bottleneck──► rl_token (d=128)
                                       │
                              [Actor MLP] ← vla_action_chunk
                                       │
                              delta_action  (tanh-bounded)
                                       │
                     final = vla_action + delta
```

### Key Components

| File | Purpose |
|------|--------|
| `rlt/models/vla_backend.py` | Abstract `VLABackend` interface — swap VLAs by implementing this |
| `rlt/models/smolvla_wrapper.py` | SmolVLA with PyTorch hooks on VLM layers; `MockVLAWrapper` for tests |
| `rlt/models/rlt_encoder.py` | Encoder-decoder bottleneck transformer |
| `rlt/models/actor.py` | SAC actor: `[rl_token, vla_action]` → tanh delta |
| `rlt/models/critic.py` | Twin-Q SAC critic |
| `rlt/training/sac.py` | SAC with reference regularization + MPS cache management |
| `rlt/training/trainer.py` | Two-phase trainer (offline pretraining + online SAC) |
| `rlt/training/replay_buffer.py` | Circular buffer with human demo injection |
| `rlt/envs/robot_env.py` | `SO101Env` (real hardware) + `MockRobotEnv` (testing) |
| `rlt/utils/device.py` | Device detection: auto-selects cuda > mps > cpu |

### Training Phases

1. **Phase 1 — Offline Bottleneck Pre-training** (`scripts/pretrain_rlt.py`):  
   Load LeRobot demos → extract SmolVLA hidden states → train encoder-decoder (MSE reconstruction). Freeze encoder after.

2. **Phase 2 — Online SAC** (`scripts/train.py`):  
   Run SAC on SO-101. Only actor + critic update. SmolVLA + encoder frozen.

---

## Repo Structure

```
vla-rlt/
├── CLAUDE.md
├── README.md
├── pyproject.toml
├── setup.py
├── rlt/                          ← main package (VLA-agnostic)
│   ├── models/
│   │   ├── vla_backend.py        # Abstract VLA interface
│   │   ├── smolvla_wrapper.py    # SmolVLA + MockVLAWrapper
│   │   ├── rlt_encoder.py        # Bottleneck encoder-decoder
│   │   ├── actor.py              # SAC actor (action-editing MLP)
│   │   └── critic.py             # Twin-Q critic
│   ├── training/
│   │   ├── sac.py                # MPS-aware SAC
│   │   ├── replay_buffer.py      # Off-policy buffer
│   │   └── trainer.py            # Two-phase training orchestrator
│   ├── envs/
│   │   └── robot_env.py          # SO101Env + MockRobotEnv + DemoDataset
│   └── utils/
│       ├── device.py             # cuda > mps > cpu auto-detection
│       ├── logging.py            # WandB + console
│       └── checkpointing.py      # Save/load with rotation
├── configs/
│   ├── default.yaml              # Tuned for M2 Air 8GB + SmolVLA + SO-101
│   └── tasks/
│       ├── so101_pick_place.yaml
│       ├── ethernet_insertion.yaml
│       └── screwdriver_alignment.yaml
├── scripts/
│   ├── pretrain_rlt.py
│   ├── train.py
│   └── evaluate.py
└── tests/
    ├── test_models.py
    └── test_training.py
```

The old `groot_rlt/` directory has been removed.

---

## Environment Setup

**Always use `uv` for dependency management.** Never use `pip` directly.

```bash
# Install uv
curl -Ls https://astral.sh/uv/install.sh | sh

# Clone and install
git clone https://github.com/akashspacesky/vla-rlt
cd vla-rlt
uv venv .venv --python 3.10
source .venv/bin/activate
uv pip install -e ".[dev]"

# For real SO-101 work (installs LeRobot):
uv pip install -e ".[dev,robot]"
# or: uv pip install lerobot
```

### Adding / removing dependencies
```bash
# Edit pyproject.toml, then:
uv pip install -e ".[dev]"
```

---

## Development Guidelines

### Running Experiments

```bash
# Validate shapes — no hardware, no SmolVLA weights needed:
python scripts/pretrain_rlt.py --dry_run

# Run tests (MPS auto-detected):
pytest tests/ -v

# Phase 1: pre-train on demos
python scripts/pretrain_rlt.py \
    --config configs/default.yaml \
    --data_dir /path/to/lerobot_demos

# Phase 2: online SAC (mock, no robot):
python scripts/train.py --mock

# Phase 2: real SO-101:
python scripts/train.py \
    --config configs/tasks/so101_pick_place.yaml \
    --encoder_ckpt checkpoints/bottleneck_pretrained.pt
```

### Key Hyperparameters (M2 Air tuned)

- `rlt_dim: 128` — Smaller than GR00T version; 8GB constraint
- `batch_size: 16` — MPS-safe
- `updates_per_step: 50` — Reduced from 200; MPS is slower than CUDA

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akashspacesky/vla-rlt](https://github.com/akashspacesky/vla-rlt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

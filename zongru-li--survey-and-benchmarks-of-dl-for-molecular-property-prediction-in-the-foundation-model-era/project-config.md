---
trigger: always_on
description: Guidelines for AI coding agents working on KA-GNN (Kolmogorov-Arnold Graph Neural Networks).
---

# AGENTS.md - KA-GNN Project Guidelines

Guidelines for AI coding agents working on KA-GNN (Kolmogorov-Arnold Graph Neural Networks).

## Project Structure

```
molecule/
├── src/
│   ├── models/         # Neural network implementations
│   │   ├── __init__.py # Model registry (MODEL_REGISTRY, get_model)
│   │   ├── base.py     # BaseModel abstract class
│   │   ├── ka_gnn.py   # KA-GNN implementations
│   │   └── ...         # Other model files
│   ├── utils/
│   │   ├── config.py   # Config loading, device/seed setup
│   │   ├── data.py     # Dataset loading, DGL graph construction
│   │   ├── pyg_data.py # PyG dataloader for PyG models
│   │   ├── training.py # Training loops, loss functions
│   │   ├── checkpoint.py # Model checkpointing
│   │   ├── output.py   # Results writing
│   │   ├── graph.py    # Molecule to graph conversion
│   │   └── splitters.py # Dataset splitting strategies
│   └── run.py          # Main entry point
├── configs/
│   ├── common.py       # Global settings (device, seed, cuda)
│   └── *.yaml          # Per-model config files
├── scripts/            # Shell scripts for experiments
├── data/               # Raw datasets and processed cache
├── output/             # Training results (CSV)
└── tmp/checkpoints/    # Saved model checkpoints
```

## Build/Lint/Test Commands

### Environment Setup
```bash
pip install -r requirements.txt
```

### Running Experiments

```bash
# Smoke test (quick validation, 2 epochs)
python src/run.py --config configs/ka_gnn.yaml --dataset bace --epochs 2

# Standard experiment
python src/run.py --config configs/ka_gnn.yaml --dataset bace --epochs 100

# Full experiment with all overrides
python src/run.py --config configs/ka_gnn.yaml --dataset bace --epochs 501 --lr 0.0001 --batch-size 128 --split scaffold --device cuda

# CPU-only run
python src/run.py --config configs/ka_gnn.yaml --dataset bace --epochs 2 --device cpu

# With checkpoint saving
python src/run.py --config configs/ka_gnn.yaml --dataset bace --save-checkpoint

# With standard deviation output
python src/run.py --config configs/ka_gnn.yaml --dataset bace --stddev
```

### CLI Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--config` | Yes | Path to YAML config file |
| `--dataset` | Yes | Dataset name |
| `--epochs` | No | Override training epochs |
| `--lr` | No | Override learning rate |
| `--batch-size` | No | Override batch size |
| `--device` | No | cuda or cpu |
| `--seed` | No | Random seed |
| `--split` | No | Split type: random, scaffold, umap, butina, time |
| `--save-checkpoint` | No | Save best model checkpoint |
| `--stddev` | No | Include std dev in output |

### Linting

```bash
ruff check .
ruff check . --fix  # Auto-fix issues
```

## Datasets

### Classification (ROC-AUC metric)
| Dataset | Tasks | Description |
|---------|-------|-------------|
| bace | 1 | BACE-1 inhibitor |
| bbbp | 1 | Blood-brain barrier penetration |
| hiv | 1 | HIV inhibition |
| clintox | 2 | Clinical toxicity |
| tox21 | 12 | Toxicity targets |
| muv | 17 | Maximum unbiased validation |
| sider | 27 | Side effects |

### Regression (PearsonR metric)
| Dataset | Tasks |
|---------|-------|
| adme_hlm | 1 |
| adme_rlm | 1 |
| adme_mdr1 | 1 |
| adme_sol | 1 |
| adme_hppb | 1 |
| adme_rppb | 1 |

## Model Registry

Models are registered in `src/models/__init__.py`:

```python
from src.models import get_model, MODEL_REGISTRY

model = get_model(config)  # Raises ValueError if unknown
```

### Model Categories

**DGL GNN Models** (use `create_dataloader` with `model_type='gnn'`):
- `ka_gnn`, `ka_gnn_two`, `mlp_sage`, `mlp_sage_two`, `kan_sage`, `kan_sage_two`
- `dmpnn`, `attentivefp`, `mol_gdl`
- `ngram_rf`, `ngram_xgb` (non-neural baselines)

**DGL GAT Models** (use `create_dataloader` with `model_type='gat'`):
- `kagat`, `mlpgat`, `kangat`, `pogat`

**PyG Models** (use `create_pyg_dataloader`):
- `pretrain_gnn`, `graphmvp`, `molclr_gcn`, `molclr_gin`
- `graphkan`, `gin`, `gcn`, `grover`, `cd_mvgnn`

## Configuration

### YAML Config Structure

```yaml
model_select: "ka_gnn"      # Model name (required)
force_field: 'uff'          # Molecular force field
encoder_atom: "cgcnn"       # Atom encoder
encoder_bond: "dim_14"      # Bond encoder
pooling: 'avg'              # Pooling: avg, sum, max, attention
loss_sclect: 'bce'          # Loss: bce, l1, l2, sml1
grid_feat: 1                # Grid features for KAN layers
num_layers: 4               # Number of GNN layers
LR: 0.0001                  # Learning rate
NUM_EPOCHS: 501             # Training epochs
batch_size: 128             # Batch size
train_ratio: 0.8            # Training split
vali_ratio: 0.1             # Validation split
test_ratio: 0.1             # Test split
iter: 1                     # Number of iterations
```

### Common Settings (`configs/common.py`)

```python
device = "cuda"
seed = 42
log_level = "INFO"
cuda = {
    "use_cuda": True,
    "device_id": 0,
    "deterministic": True,
    "benchmark": False,
}
```

## Code Style Guidelines

### Import Order

```python
# 1. Standard library (alphabetical)
import argparse
import os
import random
import sys
from pathlib import Path
from typing import Dict, Any, Optional, Tuple, List


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zongru-Li/Survey-and-Benchmarks-of-DL-for-Molecular-Property-Prediction-in-the-Foundation-Model-Era](https://github.com/Zongru-Li/Survey-and-Benchmarks-of-DL-for-Molecular-Property-Prediction-in-the-Foundation-Model-Era) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->

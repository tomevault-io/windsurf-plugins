---
trigger: always_on
description: SPICE is a framework for discovering symbolic cognitive mechanisms from behavioral data. It combines Recurrent Neural Networks (RNNs) with Sparse Identification of Nonlinear Dynamics (SINDy) to extract interpretable mathematical equations that describe latent cognitive processes.
---

# SPICE - Sparse and Interpretable Cognitive Equations

## Project Overview

SPICE is a framework for discovering symbolic cognitive mechanisms from behavioral data. It combines Recurrent Neural Networks (RNNs) with Sparse Identification of Nonlinear Dynamics (SINDy) to extract interpretable mathematical equations that describe latent cognitive processes.

## Abstract

Discovering computational models that explain human cognition and behavior remains  a  central  goal  of  cognitive  science,  yet the  reliance  on  hand-crafted equations  limits  the  range  of  cognitive  mechanisms  that  can  be  uncovered. We introduce SPICE (Sparse and Interpretable Cognitive Equations), a framework that automates the discovery of mechanistically interpretable cognitive models directly from behavioral data. SPICE fits recurrent neural networks to capture latent cognitive dynamics and then applies sparse equation discovery to extract concise mathematical expressions describing those dynamics. Theory-guided priors make the approach data- and compute-efficient, while a hierarchical design reveals individual differences in the algorithmic structure of cognitive dynamics rather than in parameters alone. In simulations, SPICE accurately recovered the structure  and  parameters  of  known  reinforcement  learning  models. Applied to human behavior in a two-armed bandit task, it uncovered new equations that outperformed existing models and revealed structural alterations in reinforcement learning mechanisms among participants with depression, such as a loss of nonlinear exploration dynamics regulating behavioral flexibility. This approach provides systematic insights into structural individual differences in cognitive mechanisms and establishes a foundation for automated discovery of interpretable behavioral models.

### Core Methodology

1. **RNN Training**: A task-specific RNN learns to predict human behavior, implicitly capturing latent cognitive mechanisms in disentangled submodules
2. **SINDy Regularization**: During training, SINDy equations act as regularizers (similar to SINDy-SHRED), pushing submodule dynamics toward spaces amenable to SINDy candidate terms
3. **Equation Discovery**: SINDy approximates the fitted dynamics in each disentangled submodule, yielding interpretable symbolic equations

## Repository Structure

```
SPICE/
├── spice/                              # Core framework (backend / pip package)
│   ├── resources/
│   │   ├── estimator.py                # SpiceEstimator — scikit-learn compatible wrapper
│   │   ├── model.py                    # BaseModel — core RNN + SINDy architecture
│   │   ├── spice_utils.py              # SpiceConfig, SpiceDataset, SpiceSignals
│   │   ├── spice_training.py           # Two-stage training pipeline
│   │   └── sindy_differentiable.py     # Differentiable SINDy polynomial library
│   ├── precoded/                       # Pre-built cognitive model architectures
│   │   ├── rescorlawagner.py           # Rescorla-Wagner learning model
│   │   ├── choice.py                   # Choice perseveration
│   │   ├── forgetting.py               # Forgetting mechanisms
│   │   ├── learningrate.py             # Dynamic learning rates
│   │   ├── interaction.py              # Interaction effects
│   │   ├── embedding.py                # Participant embeddings
│   │   ├── ddm.py                      # Drift Diffusion Model (within-trial dynamics)
│   │   ├── workingmemory.py            # Working memory with reward/choice buffers
│   │   └── workingmemory_*.py          # Working memory variants
│   └── utils/
│       ├── convert_dataset.py          # CSV ↔ SpiceDataset conversion pipeline
│       └── plotting.py                 # Visualization utilities
│
├── weinhardt2026/                      # Paper-specific code (fitting, benchmarking, analyses)
│   ├── run.py                          # Main entry point for training
│   ├── studies/                        # Self-contained study directories
│   │   ├── synthetic/                  # Synthetic parameter recovery
│   │   ├── braun2018/                  # Each study has: notebook, benchmarking script,
│   │   ├── bustamante2023/             #   data/, params/, results/
│   │   ├── castro2025/
│   │   ├── dezfouli2019/
│   │   ├── ganesh2024a/
│   │   ├── huang2026/
│   │   ├── hwang2026/
│   │   └── archive/                    # Inactive studies
│   ├── analysis/                       # Cross-study analysis pipelines
│   └── utils/                          # Shared utilities (benchmarking, bandits, etc.)
│
├── docs/                               # Documentation and tutorials
├── pyproject.toml                      # Package config (autospice v0.2.0, Python >=3.11)
├── setup.py                            # Installation
└── requirements.txt                    # Core dependencies
```

## Tech Stack

- **Language**: Python 3.11+
- **ML Framework**: PyTorch (2.7+)
- **API Style**: Scikit-learn estimator interface
- **Package Name**: `autospice` (pip installable)

## Commands

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whyhardt/SPICE](https://github.com/whyhardt/SPICE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->

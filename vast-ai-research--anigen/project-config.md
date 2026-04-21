---
trigger: always_on
description: `AGENTS.md` is the shared, canonical instruction file for coding agents in this repository.
---

# Repository Guidelines

`AGENTS.md` is the shared, canonical instruction file for coding agents in this repository.
Keep common project context, commands, conventions, and gotchas here.
If `CLAUDE.md` or `CODEX.md` exists, they should stay lightweight and only contain tool-specific addenda to avoid duplicated guidance drifting out of sync.

## Project Overview

**AniGen** is a unified framework that generates animate-ready 3D assets from a single image. It produces coherent meshes with articulated skeletons and skinning weights, ready for immediate use in 3D pipelines and motion-capture-driven animation. The core representation is *S^3 Fields* (Shape, Skeleton, Skin) defined over a shared spatial domain. Built on a two-stage flow-matching pipeline, AniGen first synthesizes a sparse structural scaffold (skeleton + coarse shape) and then generates dense geometry and articulation in a structured latent space.

- **Paper**: ACM SIGGRAPH 2026 ([arXiv:2604.08746](https://arxiv.org/pdf/2604.08746))
- **License**: MIT (source code). `extensions/CUBVH/` is non-commercial / research only (NVIDIA instant-ngp derived). See `THIRD_PARTY_LICENSES.md`.

## Project Structure & Module Organization

```
.
├── anigen/                        # Main Python package
│   ├── models/                    # Model architectures (VAE, flow matching)
│   │   ├── anigen_sparse_structure_vae.py
│   │   ├── anigen_sparse_structure_flow.py
│   │   ├── anigen_structured_latent_flow.py
│   │   └── structured_latent_vae/
│   ├── modules/                   # Neural network building blocks
│   │   ├── sparse/                # Sparse 3D convolution & attention (spconv)
│   │   ├── attention/             # Standard attention mechanisms
│   │   └── transformer/           # Transformer blocks with modulation
│   ├── pipelines/                 # Inference pipelines
│   │   ├── anigen_image_to_3d.py  # Main end-to-end inference pipeline
│   │   └── samplers/              # Flow-matching samplers
│   ├── datasets/                  # Training dataset loaders
│   ├── trainers/                  # Training orchestration
│   │   ├── vae/                   # VAE trainers (skin AE, SS VAE, SLAT VAE)
│   │   └── flow_matching/         # Flow matching trainers with CFG
│   ├── representations/           # 3D data structures
│   │   ├── mesh/                  # FlexiCubes implicit surface extraction
│   │   ├── skeleton/              # Skeleton with bone grouping
│   │   ├── gaussian/              # Gaussian splatting
│   │   └── octree/                # Spatial indexing
│   ├── renderers/                 # Rendering utilities
│   └── utils/                     # Utilities (checkpoint, mesh, image, skin, etc.)
├── configs/                       # Training configuration files (JSON)
│   ├── anigen_skin_ae.json        # Stage 1: Skin AutoEncoder
│   ├── ss_dae.json                # Stage 2: Sparse Structure DAE
│   ├── slat_dae.json              # Stage 3: Structured Latent DAE
│   ├── ss_flow_duet.json          # Stage 4: SS Flow Matching
│   └── slat_flow_auto.json        # Stage 5: SLAT Flow Matching
├── extensions/
│   └── CUBVH/                     # CUDA BVH extension (training only)
├── ckpts/                         # Pretrained model weights
│   ├── anigen/                    # AniGen models (ss_flow_*, slat_flow_*, *_dae)
│   ├── dinov2/                    # DINOv2 vision encoder
│   ├── dsine/                     # Surface normal estimator
│   └── vgg/                       # VGG backbone
├── assets/                        # Demo images and example outputs
├── third_parties/                 # External dependencies (dsine)
├── example.py                     # CLI inference entrypoint
├── app.py                         # Gradio web demo
├── train.py                       # Training entrypoint
├── setup.sh                       # Smart installation script
└── requirements.txt               # Python dependencies (non-CUDA)
```

## Environment Setup

### Prerequisites
- **OS**: Linux only.
- **GPU**: NVIDIA GPU with >= 18 GB VRAM. Tested on A800, RTX 3090.
- **CUDA Toolkit**: 11.8 or 12.2 (needed to compile extensions).
- **Python**: 3.10+.

### Installation Commands

Full install (creates a new virtual environment):
```bash
source ./setup.sh --new-env --all
```

With Tsinghua PyPI mirror (for users in China):
```bash
source ./setup.sh --new-env --all --tsinghua
```

Into an existing environment with PyTorch already installed:
```bash
source ./setup.sh --basic
```

Add Gradio demo dependencies:
```bash
source ./setup.sh --demo
```

Add training dependencies (builds CUBVH extension):
```bash
source ./setup.sh --train
```

**Important**: `setup.sh` must be **sourced** (`. ./setup.sh` or `source ./setup.sh`), not executed directly (`bash setup.sh` will fail to activate the environment).

The script auto-detects CUDA version and installs matching wheels for PyTorch, spconv, pytorch3d, and nvdiffrast. It prefers `uv` for fast installs and falls back to `pip`.

## Pretrained Model Weights

Weights are stored under `ckpts/`. The `ensure_ckpts()` function in `anigen/utils/ckpt_utils.py` auto-downloads missing weights from Hugging Face on first run.

Required weight directories:
- `ckpts/dinov2/` — DINOv2 vision encoder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VAST-AI-Research/AniGen](https://github.com/VAST-AI-Research/AniGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

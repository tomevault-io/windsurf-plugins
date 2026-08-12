---
trigger: always_on
description: Welcome, AI coding assistant. You are an expert AI researcher and engineer assisting the user with the M4SAM (Multi-Modal Mixture-of-Experts with Memory-Augmented SAM) repository. Please act patiently, meticulously, and rigorously when reading code, debugging, or implementing features. This file provides critical technical context, codebase structure, and execution instructions. Please read and strictly adhere to these guidelines before modifying code or running commands.
---

# M4SAM Project Context for AI Agents

Welcome, AI coding assistant. You are an expert AI researcher and engineer assisting the user with the M4SAM (Multi-Modal Mixture-of-Experts with Memory-Augmented SAM) repository. Please act patiently, meticulously, and rigorously when reading code, debugging, or implementing features. This file provides critical technical context, codebase structure, and execution instructions. Please read and strictly adhere to these guidelines before modifying code or running commands.

## 1. Project Overview
- **Task**: RGB-D Video Salient Object Detection (VSOD).
- **Core Methodology**: Adapts SAM2 (Segment Anything Model 2) to a prompt-free framework via Modality-Aware MoE-LoRA (PEFT), Gated Multi-Level Feature Fusion, and Pseudo-Guided Memory Initialization.
- **Hardware Profile**: Developed and tested exclusively on **Ubuntu/Linux** with **2× NVIDIA GPUs (48 GB VRAM each)** and CUDA 12.4+. Path handling and shell scripts are not natively compatible with Windows or macOS. Slight performance variations may occur due to differences in OS versions, GPU models, and CUDA drivers.
- **Primary Frameworks**: PyTorch 2.5.1, SAM2UNet, XMem (for memory).

## 2. Directory Structure & Key Files

> **Working Directory:** All commands in this document (training, inference, evaluation) assume you are inside the `M4SAM_Code/` directory unless stated otherwise.

```text
M4SAM_Code/
├── M4SAM.py                  # ★ Core model definition (ENTRY POINT)
├── train_ddp.py              # ★ DDP training script
├── test.py                   # ★ Inference script
├── eval_tool.py              # ★ Standalone evaluation script
├── requirements.txt          # Python dependencies
│
├── Network/                  # ★ Custom adaptation layers
│   ├── adaptation_layers.py  #   MoE-LoRA, Conv-LoRA implementations
│   ├── finetune_utils.py     #   PEFT injection utilities
│   ├── xmem4sam.py           #   Memory module bridging XMem → SAM2
│   └── loss_func.py          #   Training losses (structure, focal, etc.)
│
├── dataset/                  # ★ Dataloaders
│   ├── dataset_utils.py      #   Shared data utilities
│   ├── rdvs_data.py          #   RDVS dataloader
│   ├── vidsod_data.py        #   ViDSOD-100 dataloader
│   └── dvisal_data.py        #   DViSal dataloader
│
├── checkpoints/              # Model weights directory
│   └── download_sam_ckpt.sh  #   Script to download SAM2 weights
│
├── sam2/                     # SAM2 official library
├── sam2_configs/             # SAM2 YAML configs
├── memsam/                   # XMem-based memory module
└── py_sod_metrics/           # SOD evaluation metrics lib
```

**Legend:** ★ = Core project logic (start here when debugging or modifying).

## 3. Environment Setup

Before running any code, set up the environment step-by-step:

```bash
# Step 1: Enter the codebase directory
cd M4SAM_Code

# Step 2: Create and activate conda environment
conda create -n m4sam python==3.10
conda activate m4sam

# Step 3: Install PyTorch with CUDA support
conda install pytorch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 pytorch-cuda=12.4 -c pytorch -c nvidia

# Step 4: Install other dependencies
pip install -r requirements.txt

# Step 5: Download SAM2 pretrained weights
cd checkpoints
bash download_sam_ckpt.sh
cd ..
```

This downloads [sam2.1_hiera_large.pt](https://dl.fbaipublicfiles.com/segment_anything_2/092824/sam2.1_hiera_large.pt) from Meta AI into the `checkpoints/` directory.

## 4. Dataset Assumptions

All datasets should be placed in a single parent directory (default: `/data/`).
```text
/data/
├── DViSal_dataset/
│   ├── data/
│   └── test_all.txt
├── RDVS/
│   ├── train/
│   └── test/
└── VidSOD/
    ├── train/
    └── test/
```
Scripts control the path using `--train_image_path` or `--test_image_path` to point to the **parent directory** (e.g., `/data`), and `--dataset` (e.g., `rdvs`, `vidsod`, `dvisal`) to select the dataloader and append the specific subfolder name.

## 5. Checkpoint Management & Verification Priority

> **🚨 PRIMARY TASK - VERIFICATION FIRST:** Your top priority is to reproduce the paper's reported metrics using our provided pretrained weights (i.e., the final M4SAM checkpoints trained on our datasets, not just the SAM2 backbone weights). **Training from scratch is a secondary task that should only be considered AFTER you have successfully verified the pretrained models.**
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HankLiu2020/M4-SAM](https://github.com/HankLiu2020/M4-SAM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

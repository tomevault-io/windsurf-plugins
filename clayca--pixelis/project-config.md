---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Pixelis is a novel vision-language agent designed to reason directly within the pixel space of images and videos. The project builds upon three foundational works:
- **Pixel-Reasoner**: Provides pixel-space reasoning capabilities with visual operations
- **Reason-RFT**: Reinforcement Fine-Tuning framework for visual reasoning
- **TTRL (verl)**: Test-Time Representation Learning engine for continuous online evolution
- 
## Development Context
See @reference/ROADMAP.md for current status and next steps
Important!!!: When finish one task or one round, or even one phase in the ROADMAP.md, replace the ⚪ with the ✅.

## Repository Structure

```
Pixelis/
├── reference/               # Reference implementations from source projects
│   ├── Pixel-Reasoner/     # Base pixel-space reasoning implementation
│   │   ├── curiosity_driven_rl/  # RL training with curiosity rewards
│   │   ├── instruction_tuning/   # SFT implementation
│   │   └── onestep_evaluation/   # Evaluation scripts
│   ├── Reason-RFT/         # Reinforcement fine-tuning framework
│   │   ├── train/          # Training implementations (SFT & RL stages)
│   │   ├── eval/           # Evaluation scripts
│   │   └── scripts/        # Training execution scripts
│   └── TTRL/verl/          # Online learning framework
│       ├── trainer/        # Training orchestration
│       ├── workers/        # Distributed workers (FSDP, Megatron)
│       └── models/         # Model implementations
├── tasks/                  # Development roadmap tasks
└── ROADMAP.md             # Complete project roadmap
```

## Key Technologies & Frameworks

- **Base Models**: Qwen2.5-VL (7B), Qwen3 (8B)
- **Training Frameworks**: 
  - OpenRLHF (curiosity-driven RL)
  - LlamaFactory (SFT)
  - verl (TTRL/online learning)
- **Distributed Training**: Ray, FSDP, Megatron-LM
- **Inference**: vLLM, SGLang
- **Optimization**: GRPO (Group Relative Policy Optimization)

## Common Development Commands

### Environment Setup
```bash
# Create conda environments for different components
conda create -n pixelis-sft python=3.10
conda create -n pixelis-rl python=3.10
conda create -n pixelis-ttrl python=3.10

# Install dependencies (component-specific)
pip install -r reference/Pixel-Reasoner/curiosity_driven_rl/requirements.txt  # For RL
pip install -r reference/Reason-RFT/requirements_sft.txt                      # For SFT
pip install -r reference/TTRL/verl/requirements.txt                          # For TTRL
```

### Data Generation Commands

#### Two-Stage CoTA Data Generation
The project uses a two-stage data generation pipeline for creating Chain-of-Thought-Action (CoTA) training data:

**Stage 1: Generate Specialized Datasets**
```bash
# Generate individual task-specific datasets
python scripts/1_generate_specialized_datasets.py \
    --manifest configs/data_generation_manifest.yaml \
    --output-dir data_outputs/specialized \
    --verbose

# Dry run to check configuration
python scripts/1_generate_specialized_datasets.py \
    --manifest configs/data_generation_manifest.yaml \
    --dry-run

# Generate specific tasks only
python scripts/1_generate_specialized_datasets.py \
    --manifest configs/data_generation_manifest.yaml \
    --tasks detail_perception_task geometric_reasoning_task
```

**Stage 2: Fuse and Validate Final Datasets**
```bash
# Create final SFT and RFT datasets with augmentation
python scripts/2_fuse_and_validate_dataset.py \
    --fusion-manifest configs/data_fusion_manifest.yaml \
    --input-dir data_outputs/specialized \
    --output-dir data_outputs/final \
    --verbose

# Dry run to check fusion configuration
python scripts/2_fuse_and_validate_dataset.py \
    --fusion-manifest configs/data_fusion_manifest.yaml \
    --dry-run
```

### Training Commands

#### Supervised Fine-Tuning (SFT)
```bash
# Pixel-Reasoner SFT
cd reference/Pixel-Reasoner/instruction_tuning
bash sft.sh

# Reason-RFT SFT (with curriculum learning)
cd reference/Reason-RFT
conda activate reasonrft_sft
bash scripts/train/cot_sft/resume_finetune_qwen2vl_7b_task1_cot_sft.sh
```

#### Reinforcement Learning Training
```bash
# Pixel-Reasoner RL (curiosity-driven)
cd reference/Pixel-Reasoner/curiosity_driven_rl
bash scripts/train_vlm_single.sh  # Single node
bash scripts/train_vlm_multi.sh   # Multi-node

# Reason-RFT RL (GRPO-based)
cd reference/Reason-RFT
conda activate reasonrft_rl
bash scripts/train/reason_rft/stage_rl/resume_finetune_qwen2vl_7b_task1_stage2_rl.sh
```

#### TTRL/Online Learning
```bash
cd reference/TTRL/verl
# PPO training
bash examples/ppo_trainer/run_qwen2-7b.sh
# GRPO training  
bash examples/grpo_trainer/run_qwen2-7b.sh
```

### Evaluation Commands
```bash
# Pixel-Reasoner evaluation
cd reference/Pixel-Reasoner/curiosity_driven_rl
bash scripts/eval_vlm_new.sh

# Reason-RFT evaluation
cd reference/Reason-RFT
bash scripts/eval/open_source_models/single_gpu_eval/eval_by_vllm_task1_reason_rft_single_gpu.sh
```

### Testing
```bash
# verl tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Clayca/Pixelis](https://github.com/Clayca/Pixelis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->

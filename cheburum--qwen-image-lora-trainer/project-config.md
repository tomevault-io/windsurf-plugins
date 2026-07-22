---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Qwen-Image LoRA/DoRA training framework** built with PyTorch Lightning. It enables fine-tuning of the Qwen-Image diffusion model using LoRA (Low-Rank Adaptation) weights while keeping the base model frozen. The project implements flow matching training, aspect ratio bucketing for efficient multi-resolution training, and pre-computed text embeddings.

**Key Technologies:**
- PyTorch Lightning for distributed training orchestration
- PEFT (LoRA adapter) for parameter-efficient fine-tuning
- Diffusers library for Qwen-Image model integration
- Albumentations for image augmentation
- Optimum-Quanto for 8-bit quantization of transformer blocks

## Development Commands

### Training
```bash
# Basic training with default config
python scripts/train.py

# Custom config file
python scripts/train.py --config configs/train_config.yaml

# Command-line overrides
python scripts/train.py --data.batch_size=8 --model.lora_rank=64 --trainer.max_epochs=20

# Resume from checkpoint
python scripts/train.py --ckpt_path=./outputs/checkpoints/epoch_005/pytorch_model/adapter_model.bin
```

### Dataset Management
```bash
# Generate image captions from directory
python scripts/image_captions.py --images_dir /path/to/images --output_json captions.json

# Visualize dataset with ARB bucketing
python scripts/visualize_dataset.py --images_dir /path/to/images --json_path captions.json --bucket_output_dir visualizations/
```

### Development
```bash
# Install dependencies
pip install -r requirements.txt

# Format code (if flake8/black are used)
# Currently no formatter configured - add as needed
```

## Architecture Overview

### Core Components

#### 1. **LoRA Model (`src/models/lora_model.py:QwenImageLoRAModule`)**
- PyTorch Lightning module handling the entire training pipeline
- **Model Components:**
  - **VAE (AutoencoderKLQwenImage)**: Frozen, encodes images to latent space (vae_scale_factor=2)
  - **Transformer (QwenImageTransformer2DModel)**: Quantized to 8-bit with LoRA adapters on attention layers (to_k, to_q, to_v, to_out.0)
  - **Noise Scheduler (FlowMatchEulerDiscreteScheduler)**: Flow matching instead of DDPMs for faster convergence

- **Training Process:**
  1. Images → VAE latents (normalized with learned latents_mean/std)
  2. Random noise sampling + density-weighted timestep sampling
  3. Flow matching: predict velocity field (v = noise - data) instead of noise
  4. Weighted MSE loss with loss weighting scheme
  5. Save LoRA weights in both diffusers and PyTorch formats

- **Key Design Choices:**
  - Gradient checkpointing enabled on transformer for memory efficiency
  - Only LoRA parameters (≈0.5-1% of base model) are trainable
  - bfloat16 precision for numerical stability with large models
  - 8-bit quantization of non-LoRA transformer blocks saves ~50% VRAM

#### 2. **Dataset & Aspect Ratio Bucketing (`src/data/dataset.py`)**
Implements NovelAI's bucketing algorithm for efficient multi-resolution training:

- **Three-Phase Process:**
  1. **Initialization Phase:** Generate buckets, assign images to nearest bucket, pre-compute resize parameters
  2. **Runtime Phase:** Load images from disk, resize to bucket dimensions, apply augmentations (loaded from JSON metadata)
  3. **Batching Phase:** BucketBatchSampler ensures each batch contains only images from same bucket (same H×W)

- **Key Classes:**
  - `AspectRatioBucket`: Represents a single (width, height) bucket with aspect ratio and pixel count
  - `ImageCaptionDataset`: Loads JSON with captions, manages text embeddings cache, applies per-image augmentations
  - `BucketBatchSampler`: Creates homogeneous batches where all images have identical dimensions

- **Bucket Generation Algorithm:**
  - Start with min_resolution (256), increment by resolution_step (64)
  - For each width, find largest height where width×height ≤ target_pixel_count (1328²)
  - Repeat with roles swapped, remove duplicates, ensure base resolution included
  - Result: ~50-100 buckets covering common aspect ratios

- **Text Embeddings Cache:**
  - Pre-computed during dataset initialization using QwenImagePipeline's text encoder
  - Stored as `.pt` files in `text_embeddings_cache/` directory
  - Handles partial caches (recomputes only missing embeddings)
  - Validates captions in JSON, filters out empty/failed entries

#### 3. **Training Callbacks (`src/training/callbacks.py:CheckpointCallback`)**
- Saves LoRA checkpoints at epoch and step intervals
- Dual format output: `diffusers_model/` (for pipeline inference) and `pytorch_model/` (raw state dict)
- Default: save every epoch to `./outputs/checkpoints/epoch_NNN/`

### Configuration System

**Lightning CLI Config (`configs/train_config.yaml`):**
- Uses YAML format with class path references
- Trainer settings: accelerator, devices, precision (bf16), gradient accumulation
- Model: LoRA rank/alpha, learning rate, scheduler type, warmup steps
- Data: image directory, JSON caption path, batch size, ARB settings
- Custom callback class paths for checkpoint saving

**Configuration Override Examples:**
```yaml
# In config file
model:
  lora_rank: 32

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cheburum/qwen_image_lora_trainer](https://github.com/Cheburum/qwen_image_lora_trainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

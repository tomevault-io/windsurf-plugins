---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MegaTransformer is a multimodal autoregressive world model combining text, audio, voice, and image modalities. The project implements a recurrent transformer architecture with modality-specific VAE encoders/decoders and token interleaving for unified sequence processing.

## Commands

### Training

Training uses subcommands for different model types. All training scripts share common arguments.

```bash
# Audio CVAE (speaker-conditioned VAE with FiLM)
python -m src.scripts.train.train audio-cvae --run_name my_run --config small --cache_dir ../cached_datasets/sive_cvae_f0

# Vocoder (mel-to-waveform)
python -m src.scripts.train.train vocoder --run_name my_vocoder --config tiny --cache_dir ../cached_datasets/audio

# SIVE (Speaker-Invariant Voice Encoder with CTC + GRL)
python -m src.scripts.train.train audio-sive --run_name my_sive --config small --cache_dir ../cached_datasets/audio_sive

# Image VAE
python -m src.scripts.train.train image-vae --run_name my_image_vae --config small --cache_dir ../cached_datasets/image

# World Model (multimodal)
python -m src.scripts.train.train world --run_name my_world --config small --include_modes text,audio,image
```

Common training arguments:
- `--resume_from_checkpoint <path>`: Resume from checkpoint
- `--use_deepspeed --deepspeed_config ds_config.json`: Enable DeepSpeed
- `--bf16` / `--fp16`: Mixed precision training
- `--use_gradient_checkpointing`: Memory optimization
- `--use_gan`: Enable GAN training (for VAE models)
- `--use_muon`: Use Muon+AdamW optimizer (with `--lr_muon`, `--lr_adamw`)
- `--use_ema --ema_decay 0.9999`: Exponential moving average
- `--compile_model`: torch.compile the model
- `--metrics_backend tensorboard|wandb`: Logging backend (default: tensorboard)

### Data Preprocessing

```bash
# Audio preprocessing (extracts SIVE features, speaker embeddings, F0, mel specs)
python -m src.scripts.data.preprocess_dataset audio \
    --dataset_name mozilla-foundation/common_voice_17_0 \
    --dataset_config en --split train \
    --output_dir ../cached_datasets/audio_train \
    --sive_checkpoint_path ./checkpoints/sive \
    --compute_speaker_embeddings --extract_f0 --save_mel_specs

# Build shard index after preprocessing
python -m src.scripts.data.preprocess_dataset stat-shards --output_dir ../cached_datasets/audio_train

# Multi-GPU preprocessing (run on each GPU)
python -m src.scripts.data.preprocess_dataset audio --gpu_id 0 --total_gpus 4 ...
```

### Testing

```bash
pytest tests/                          # Run all tests
python -m pytest tests/ -v             # Verbose output
python -m pytest tests/test_collator_token_placement.py  # Single test file
```

Tests cover data collation logic (BO*/PH/EO* token placement, text target alignment). No linting or formatting tools are configured.

### Inference / Evaluation

```bash
# Voice cloning demo (Gradio UI) — combines SIVE + CVAE + vocoder pipeline
python -m src.scripts.eval.audio.cvae.voice_clone --sive_checkpoint_path ./checkpoints/sive --cvae_checkpoint_path ./checkpoints/cvae --vocoder_checkpoint_path ./checkpoints/vocoder
```

Eval scripts live in `src/scripts/eval/` with subdirectories per modality.

### TensorBoard

```bash
./tensorboard.sh  # or: tensorboard --logdir runs/
```

## Architecture

### Directory Structure

- `src/model/`: Neural network modules
  - `world/`: Core world model (`MegaTransformerWorldModel`, recurrent transformer, KV cache)
  - `audio/`: Audio models (VAE, SIVE conformer, vocoder)
  - `image/`: Image VAE models
  - `text/`: Text feature extractor and generator
  - `transformer.py`: `MegaTransformerBlock` with GQA, rotary embeddings, ALiBi

- `src/config/`: Dataclass configs with predefined configurations (small, medium, large)
  - Each model has `*_CONFIGS` dicts mapping config names to dataclass instances
  - `common.py`: `MegaTransformerBlockConfig` shared across models

- `src/scripts/train/`: Training scripts
  - `train.py`: Main entry point with subcommand routing
  - `trainer.py`: `CommonTrainer` base class extending HuggingFace Trainer
  - `optimizers.py`: `MuonAdamW` custom optimizer
  - `audio/vae/`, `audio/vocoder/`, `audio/sive/`, `image/vae/`, `world/`: Model-specific trainers

- `src/scripts/data/`: Dataset preprocessing and loading
  - `preprocess_dataset.py`: Main preprocessing entry point with modality-specific `Preprocessor` subclasses
  - `audio/`, `image/`, `text/`, `world/`: Per-modality dataset, collator, and preprocessor implementations

- `src/utils/`: Shared utilities
  - `metrics.py`: Central metrics logging module (backend-agnostic singleton)
  - `metrics_backend.py`: `MetricsBackend` protocol, `TensorBoardBackend`, `NoOpBackend`
  - `wandb_backend.py`: `WandBBackend` with context-aware media grouping
  - `visualization.py`: Pure rendering functions (mel specs, attention weights, vocoder audio)
  - `model_loading_utils.py`: `load_model()` function for loading from config + checkpoint
  - `audio_utils.py`: `SharedWindowBuffer` for efficient STFT/mel computation
  - `speaker_encoder.py`: ECAPA-TDNN and WavLM speaker encoders

### Key Design Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mekadrom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

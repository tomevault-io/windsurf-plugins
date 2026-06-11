---
trigger: always_on
description: **PCRL-MRG** is the official code for the EMNLP 2024 paper:
---

# CLAUDE.md — AI Assistant Guide for PCRL-MRG

## Project Overview

**PCRL-MRG** is the official code for the EMNLP 2024 paper:
> "See Detail Say Clear: Towards Brain CT Report Generation via Pathological Clue-driven Representation Learning"

The project implements an automated **brain CT report generation** system using a multimodal LLM (LLaMA 3) fine-tuned on the **CTRG** (CT Report Generation) dataset. Reports are generated in **Chinese**.

Dataset source: https://github.com/tangyuhao2016/CTRG

---

## Repository Structure

```
PCRL-MRG/
├── README.md
├── LICENSE
│
├── data_peparation/               # [NOTE: typo in directory name, not "preparation"]
│   ├── segment_anything/          # Meta SAM model (Segment Anything Model)
│   │   ├── modeling/              # SAM architecture (encoder, decoder, transformer)
│   │   ├── utils/                 # SAM utilities (amg, onnx, transforms)
│   │   ├── automatic_mask_generator.py
│   │   ├── build_sam.py
│   │   └── predictor.py
│   ├── get_visual_feature/        # Feature extraction scripts
│   │   ├── clip_vit_1024.py       # Extracts CLIP-ViT-Large-patch14 features → .npz
│   │   └── resnet101_2048.py      # Extracts ResNet-101 features → .npz
│   ├── step1_sam_gen_masks.py     # Pipeline step 1: SAM mask generation
│   ├── step2_re_extract_entities.py  # Step 2: Entity extraction & dataset splits
│   ├── step3_clip_select_masks.py    # Step 3: CLIP-based mask selection per entity
│   └── step4_merge_entities_per_img.py  # Step 4: Merge entity annotations per image
│
├── dataset/                       # Dataset annotation files (no raw images)
│   ├── CTRG_SAM_SEG_dataset/
│   │   ├── seg_mask_anno_train.json   # SAM segmentation annotations (train)
│   │   └── splits/                    # JSON split files
│   │       ├── train.json
│   │       ├── validation.json
│   │       └── test.json
│   ├── origin_anno.json               # Original CTRG annotations
│   └── total_image_list_final.json    # Maps sample_id → list of 24 CT image paths
│
├── llama_recipes/                 # Main training framework (adapted from Meta's llama-recipes)
│   ├── configs/                   # Training configuration dataclasses
│   │   ├── training.py            # train_config (main config)
│   │   ├── peft.py                # LoRA / llama_adapter / prefix configs
│   │   ├── fsdp.py                # FSDP distributed training config
│   │   ├── datasets.py            # Dataset config dataclasses (ctrg_dataset)
│   │   └── wandb.py               # W&B logging config
│   ├── models/                    # Model implementations
│   │   ├── PCRL_llama/            # Main paper contribution: PCRL model
│   │   │   └── modeling_PCRL_llama.py
│   │   ├── ViT_MLP_llama/         # Simpler LLaVA-style baseline
│   │   │   └── modeling_ViT_MLP_llama.py
│   │   └── llama/                 # LLaMA base model code
│   │       ├── configuration_llama.py
│   │       ├── modeling_llama.py
│   │       ├── tokenization_llama.py
│   │       └── tokenization_llama_fast.py
│   ├── datasets/                  # Dataset loaders
│   │   └── ctrg_dataset.py        # CTRG dataset loader & tokenization
│   ├── data/                      # Data utilities
│   │   ├── concatenator.py        # ConcatDataset for packing strategy
│   │   ├── sampler.py             # Distributed samplers
│   │   └── llama_guard/           # Llama Guard fine-tuning utilities
│   ├── policies/                  # Training policies
│   │   ├── activation_checkpointing_functions.py
│   │   ├── anyprecision_optimizer.py
│   │   ├── mixed_precision.py
│   │   └── wrapping.py
│   ├── utils/                     # Training/evaluation utilities
│   │   ├── train_utils.py         # Training loop, eval, setup functions
│   │   ├── test_utils.py          # test_conditional_generation()
│   │   ├── config_utils.py        # update_config(), generate_peft_config()
│   │   ├── dataset_utils.py       # get_preprocessed_dataset()
│   │   ├── memory_utils.py        # MemoryTrace context manager
│   │   ├── flop_utils.py          # FlopMeasure for throughput profiling
│   │   ├── plot_metrics.py        # Metrics plotting
│   │   └── hf_llama_conversion/   # HuggingFace weight conversion utilities
│   └── model_checkpointing/       # Checkpoint save/load handlers
│       └── checkpoint_handler.py
│
└── scripts/                       # Entry-point scripts
    └── VIT_MLP_llama/
        ├── finetuning_ViT_MLP_llama.py  # Training entry point
        ├── test_ViT_MLP_llama.py        # Testing entry point
        ├── finetuning_CTRG_MRG.sh       # Training shell script (paths need updating)
        └── test_CTRG_MRG.sh             # Testing shell script (paths need updating)
```

---

## Architecture

### Two Models Are Implemented

#### 1. PCRL-LLaMA (Main Contribution) — `llama_recipes/models/PCRL_llama/`
The full PCRL model described in the paper:
- **Visual encoder**: ResNet-101 CNN producing 2048-dim features from 24 CT slices (14×14 spatial grid)
- **Segmentation head**: Per-image segmentation map generation
- **Text encoder**: BERT for encoding pathological entity descriptions
- **Contrastive learning**: CLIP-style loss aligning image segments with entity text descriptions
- **Language model**: LLaMA 3-8B-Instruct with INT4 quantization + LoRA

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chauncey-Jheng/PCRL-MRG](https://github.com/Chauncey-Jheng/PCRL-MRG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->

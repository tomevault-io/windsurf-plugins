---
trigger: always_on
description: This document is designed to guide AI coding agents through the structure,
---

# Hackable Diffusion Adapter — Agent Playbook

This document is designed to guide AI coding agents through the structure,
setup, testing, and training workflows of the **Hackable Diffusion (HD) Text
Diffusion Supervised Fine-Tuning (SFT)** adapter library.

---

## 📋 Codebase Structure

The project is structured as a standard Python/JAX package.

*   **`configs/`**: Kauldron config files defining task hyperparameters, datasets, losses, optimizers, and evaluators.
    *   [`sft_sudoku.py`](gemma/diffusion/hackable_diffusion_adapter/configs/sft_sudoku.py): LoRA-based SFT training for the Sudoku puzzle solving task.
    *   [`sft_sudoku_full.py`](gemma/diffusion/hackable_diffusion_adapter/configs/sft_sudoku_full.py): Full weight SFT training for the Sudoku puzzle solving task (no LoRA).
    *   [`sft_pubmedqa.py`](gemma/diffusion/hackable_diffusion_adapter/configs/sft_pubmedqa.py): LoRA-based SFT training for the PubMedQA long-answer task.
*   **`data/`**: Dataset loading, custom pipelines, and preprocessing transforms.
    *   [`data.py`](gemma/diffusion/hackable_diffusion_adapter/data/data.py): Common transforms (e.g. `CanvasChunker` for localized diffusion).
*   **`hd/`**: Core modeling, network layers, and state handling.
    *   [`sft_model.py`](gemma/diffusion/hackable_diffusion_adapter/hd/sft_model.py): Core `SFTDiffusion` class managing the hybrid AR prefill and localized diffusion denoising steps.
    *   [`lora.py`](gemma/diffusion/hackable_diffusion_adapter/hd/lora.py): PEFT LoRA wrappers.
    *   [`mask_helpers.py`](gemma/diffusion/hackable_diffusion_adapter/hd/mask_helpers.py): Right-pad causal/block masks and cursor tracking.
*   **`eval/`**: Custom evaluation metrics designed to avoid TPU/GPU OOM issues.
    *   [`sudoku_eval.py`](gemma/diffusion/hackable_diffusion_adapter/eval/sudoku_eval.py): Unified host-side `SudokuAllMetrics` evaluation.

---

## 🛠️ Setup and Installation

Follow these instructions to set up the local Python environment.

### 1. Python Environment Setup
We recommend Python 3.12 and CUDA 13.

### 2. Installation
First install the gemma package.

**From PyPI (Recommended)**

```bash
pip install gemma
```

**From Source**

```bash
git clone https://github.com/google-deepmind/gemma.git
cd gemma
pip install .
```

Then we additionally require `jax[cuda13]` dependencies that can be installed
via

```bash
pip install -U jax[cuda13]
```

> **CUDA Version Constraint**: If configuring JAX for GPU, you must use
> **CUDA 13**. Mixing CUDA 12 packages (such as `jax-cuda12-plugin` or
> `nvidia-nccl-cu12`) will trigger PJRT initialization crashes and silent
> NCCL corruption.

---

## 💾 Dataset Preparation

Before launching SFT training, datasets must be preprocessed.
Note that running these scripts requires first cloning the source repository.

### PubMedQA Dataset
```bash
cd gemma/diffusion/hackable_diffusion_adapter/data/pubmedqa
bash prepare_pubmedqa_dataset.sh
cd -
```

### Sudoku Dataset
Requires Kaggle API access token. Ask the user to generate an access token and
then run the following command.

```bash
mkdir -p ~/.kaggle && echo YOUR_KAGGLE_TOKEN > ~/.kaggle/access_token && chmod 600 ~/.kaggle/access_token
```

Then the datapipeline can be run with the following command

```bash
cd gemma/diffusion/hackable_diffusion_adapter/data/sudoku
bash prepare_sudoku_dataset.sh
cd -
```

---

## 🧪 Running Unit Tests

To verify the JAX layers, data pipelines, and sampling routines, run:

```bash
pytest gemma/diffusion/hackable_diffusion_adapter/
```

Or run individual tests:
```bash
pytest gemma/diffusion/hackable_diffusion_adapter/hd/lora_test.py
```

---

## 🚀 Launching SFT Training

Always use the standard Kauldron CLI command. Use the following env variables
to prevent JIT compilation OOMs and NCCL communication hangs:

Launches should be run from the parent dir of the gemma directory.

#### PubMedQA

```bash
env XLA_FLAGS="--xla_disable_hlo_passes=constant_folding" \
    NCCL_ALGO="Ring" \
    NCCL_PROTO="LL128" \
    NCCL_NVLS_ENABLE="0" \
    NCCL_CUMEM_ENABLE="0" \
    python3 -m kauldron.main \
  --cfg=gemma/diffusion/hackable_diffusion_adapter/configs/sft_pubmedqa.py \
  --cfg.workdir=$(pwd)/xp_dir
```

#### Sudoku (with LoRA)

```bash
env XLA_FLAGS="--xla_disable_hlo_passes=constant_folding" \
    NCCL_ALGO="Ring" \
    NCCL_PROTO="LL128" \
    NCCL_NVLS_ENABLE="0" \
    NCCL_CUMEM_ENABLE="0" \
    python3 -m kauldron.main \
  --cfg=gemma/diffusion/hackable_diffusion_adapter/configs/sft_sudoku.py \
  --cfg.workdir=$(pwd)/xp_dir
```

#### Sudoku (full weight updates)

```bash
env XLA_FLAGS="--xla_disable_hlo_passes=constant_folding" \
    NCCL_ALGO="Ring" \
    NCCL_PROTO="LL128" \
    NCCL_NVLS_ENABLE="0" \
    NCCL_CUMEM_ENABLE="0" \
    python3 -m kauldron.main \
  --cfg=gemma/diffusion/hackable_diffusion_adapter/configs/sft_sudoku_full.py \
  --cfg.workdir=$(pwd)/xp_dir
```

---

## 📊 Offline Evaluation

Evaluation is run **offline** — it is a separate step from training. The
`eval_main` binary loads a saved checkpoint, runs AR diffusion sampling
on the eval dataset, and reports task-specific metrics.

### Running an Eval Job

From the parent dir of the gemma directory:

```bash
env XLA_FLAGS="--xla_disable_hlo_passes=constant_folding" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google-deepmind/gemma](https://github.com/google-deepmind/gemma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

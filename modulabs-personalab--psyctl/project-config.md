---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PSYCTL is a Python CLI tool for LLM personality steering using Contrastive Activation Addition (CAA) and Bidirectional Preference Optimization (BiPO). The tool enables extracting steering vectors from models to modify personality traits during text generation.

**Core Workflow:**
1. **Dataset Generation**: Create contrastive personality datasets using P2 personality prompts
2. **Vector Extraction**: Extract steering vectors using mean_diff or BiPO methods
3. **Steering Application**: Apply vectors to model activations during inference
4. **Benchmarking**: Test personality changes using psychological inventories (IPIP-NEO, NPI-40, MACH-IV, etc.)

## Development Commands

### Environment Setup
```powershell
# Install uv package manager
Invoke-WebRequest -Uri "https://astral.sh/uv/install.ps1" -OutFile "install_uv.ps1"
& .\install_uv.ps1

# Create and activate virtual environment
uv venv
& .\.venv\Scripts\Activate.ps1

# Install dependencies
uv sync

# Install development dependencies
& .\scripts\install-dev.ps1
```

### Common Development Tasks

**Pre-commit hooks** (automatic code quality checks):
```powershell
# Install hooks (one-time setup)
pre-commit install

# Run manually on all files
pre-commit run --all-files
```

**Format code** (must run before commits):
```powershell
& .\scripts\format.ps1
# Runs: ruff check --fix src/ && ruff format src/
```

**Run tests**:
```powershell
& .\scripts\test.ps1
# Runs: pytest -v --cov=psyctl --cov-report=html
```

**Run single test**:
```powershell
uv run pytest tests/test_core/test_layer_accessor.py -v
uv run pytest tests/test_core/test_layer_accessor.py::test_get_layer_basic -v
```

**Complete build** (format + lint + type check + test + install):
```powershell
& .\scripts\build.ps1
# Runs: ruff check --fix, ruff format, pyright, pytest, pip install -e .
```

**Linting and Type Checking**:
```powershell
uv run ruff check src/
uv run pyright src/
```

### Running PSYCTL Commands

**Set required environment variable**:
```powershell
$env:HF_TOKEN = "your_huggingface_token_here"
```

**Generate steering dataset**:
```powershell
psyctl dataset.build.steer --model "gemma-3-270m-it" --personality "Extroversion" --output "./dataset/steering" --limit-samples 100
```

**Extract steering vector**:
```powershell
psyctl extract.steering --model "gemma-3-270m-it" --layer "model.layers[13].mlp.down_proj" --dataset "./dataset/steering" --output "./steering_vector/out.safetensors"
```

**Apply steering**:
```powershell
psyctl steering --model "gemma-3-270m-it" --steering-vector "./steering_vector/out.safetensors" --input-text "Tell me about yourself"
```

## Architecture

### High-Level Structure

```
CLI Layer (cli.py)
    ↓
Commands Layer (commands/)
    ↓
Core Layer (core/)
    ├── Dataset Generation (dataset_builder.py, prompt.py)
    ├── Vector Extraction (steering_extractor.py, extractors/)
    ├── Steering Application (steering_applier.py)
    └── Infrastructure (hook_manager.py, layer_accessor.py)
    ↓
Models Layer (models/)
    ├── LLM Loading (llm_loader.py)
    ├── Vector Storage (vector_store.py)
    └── API Clients (openrouter_client.py)
```

### Key Components

**1. Dataset Builder (`core/dataset_builder.py`)**
- Generates contrastive personality datasets from conversation data (allenai/soda)
- Uses P2 class to create personality prompts ("Extroversion" → detailed personality description)
- Creates positive/neutral response pairs for each situation
- Supports local models and OpenRouter API
- Uses Jinja2 templates for roleplay prompts (`templates/roleplay_prompt.j2`)
- Batch processing with checkpoint support for resumable generation
- Output: JSONL files with `situation`, `char_name`, `positive`, `neutral` fields

**2. Steering Extractor (`core/steering_extractor.py`)**
- Coordinates extraction using pluggable extractor classes
- Three extraction methods:
  - **mean_diff** (`extractors/mean_difference.py`): Computes mean activation difference between positive/neutral responses
  - **denoised_mean_diff** (`extractors/denoised_mean_difference.py`): PCA-based denoising for noise reduction and improved robustness (variance threshold: 0.95)
  - **bipo** (`extractors/bipo.py`): Bidirectional Preference Optimization using DPO loss
- Layer specification via string paths (e.g., `"model.layers[13].mlp.down_proj"`)
- Uses `LayerAccessor` for dynamic layer access
- Uses `ActivationHookManager` to collect activations via PyTorch forward hooks
- Output: safetensors files with embedded metadata

**3. Hook Manager (`core/hook_manager.py`)**
- Manages PyTorch forward hooks for activation collection
- Accumulates activations across batches with incremental mean computation
- Handles padding by using attention masks to exclude padded tokens
- Thread-safe activation storage
- `register_hooks()` → run inference → `get_mean_activations()` → `remove_all_hooks()`

**4. Layer Accessor (`core/layer_accessor.py`)**
- Dynamically accesses model layers via string paths
- Supports bracket indexing: `"model.layers[13].mlp.down_proj"`
- Handles different model architectures (LLaMA, Gemma, Qwen, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modulabs-personalab/psyctl](https://github.com/modulabs-personalab/psyctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->

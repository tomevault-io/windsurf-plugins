---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

AgentFlux is a framework for optimizing LLM tool-calling through **DualTune**, a dual-stage finetuning approach that separates tool classification from argument generation. The system consists of three main components:

1. **DualTune Finetuning Pipeline** (`finetune/`) - Generates synthetic training data and trains specialized models
2. **AgentFlux Inference System** (`inference/agentflux/`) - FastAPI proxy that routes requests through finetuned models
3. **Rena Core Orchestration** (`orchestration_framework/`) - Rust-based evaluation framework with Python MCP runtime

## Key Architecture Principles

### DualTune Two-Stage Approach

The core innovation is **decoupled optimization**:
- **Stage 1**: A lightweight **classifier model** selects which tool to use (voting across 10 samples at temperature=1.0)
- **Stage 2**: Per-tool **adapter models** generate precise arguments for the selected tool

This separation allows each model to specialize rather than forcing a single model to handle both routing and argument generation.

### Tool Categories

The system is organized by **categories** (e.g., `filesys`, `monday`, `notion`). Each category contains:
- Multiple related tools (e.g., filesys has `read_file`, `write_file`, `list_directory`, etc.)
- One shared classifier model for all tools in the category
- Individual adapter models for each tool

When adding support for new tools, you typically work within a category context.

## Common Commands

### Finetuning Pipeline

```bash
cd finetune

# Full pipeline for a category (query gen → trajectory collection → data cleaning → training)
bash scripts/finetune.sh <category>

# Train only the classifier
bash scripts/finetune_classifier.sh <category> [batch_size] [grad_accum] [epochs]
# Default: batch_size=4, grad_accum=4, epochs=4

# Train only the tool adapters
bash scripts/finetune_tool_adaptors.sh <category> [batch_size] [grad_accum] [epochs]

# Generate synthetic queries (requires OPENAI_API_KEY)
python gen_queries.py --category <category> --output_path <path>

# Collect trajectories from baseline model (requires OPENAI_API_KEY and WORKSPACE env var)
WORKSPACE=/path/to/workspace python gen_trajs.py \
  --category <category> \
  --model gpt-5-mini \
  --input_queries <queries.txt> \
  --output_trajs <trajs.jsonl>

# Prepare data (clean, validate, split into train/eval/test)
python data_prepare.py --category <category>

# Generate tool-specific chat templates
python gen_tool_template.py --category <category> --model_folder <base_model_path>
```

**Training outputs**:
- Models: `finetune/<category>/results/finetune_output/`
- Epoch 2 checkpoints (used for serving): `finetune/<category>/results/finetune_serve/`
- Logs: `finetune/<category>/results/log/`
- Data: `finetune/<category>/results/trajectories/`

### Inference System

```bash
cd inference/agentflux

# Start vLLM server with LoRA adapters (edit vllm.sh to configure category and paths)
bash scripts/vllm.sh &

# Start AgentFlux proxy (defaults to port 9015)
bash scripts/proxy.sh [category]
# Default category: filesys

# The proxy exposes: http://localhost:9015/v1/chat/completions
```

The proxy is OpenAI-compatible - send requests using the OpenAI SDK with `base_url="http://localhost:9015/v1"`.

### Evaluation

```bash
cd orchestration_framework/evaluation

# Full evaluation pipeline
bash scripts/evaluate.sh <category>

# Individual steps:
# 1. Generate test queries
python gen_queries.py --category <category>

# 2. Run with AgentFlux
python run_agentflux.py <category> \
  --classifier config/<category>/classifier.json \
  --tool_adapters config/<category>/tool_adapters.json \
  --query <category>/queries/fuzzing_queries.txt \
  --output <category>/eval-results/trajs.jsonl

# 3. Run baseline comparison
python run_baseline.py <category> \
  --query <category>/queries/fuzzing_queries.txt \
  --output <category>/baseline-results/trajs.jsonl

# 4. Judge results
python <category>/judge.py \
  --trajs <category>/eval-results/trajs.jsonl \
  --output <category>/judge-results/judged.jsonl

# 5. Calculate scores
python score.py --llm_judge_path <category>/judge-results/judged.jsonl
```

### Rena Core (Rust Orchestration)

```bash
cd orchestration_framework/rena-core

# Setup (requires Python 3.11, Rust, Docker)
make setup

# Build only browserd (Rust)
make setup_browserd

# Build only runtime (Python)
make setup_runtime

# Run tests
make test

# Cleanup Docker images
make cleanup
```

## Configuration Files

Each category requires these files in `inference/agentflux/config/<category>/`:

- **`tool_list.json`**: MCP tool definitions in OpenAI format (array of tool objects)
- **`classifier.json`**: Classifier model configuration
  ```json
  {
    "model": "filesys-classifier",
    "port": 8001,
    "tools": ["read_file", "write_file", ...]
  }
  ```
- **`tool_adapters.json`**: Per-tool adapter configurations
  ```json
  {
    "read_file": {"model": "read_file-adapter", "port": 8002},
    "write_file": {"model": "write_file-adapter", "port": 8003}
  }
  ```

For finetuning, each category needs in `finetune/<category>/`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [efeslab/AgentFlux](https://github.com/efeslab/AgentFlux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

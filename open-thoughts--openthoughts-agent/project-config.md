---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Claude Code Environment Notes

**Conda Environment**: Use the otagent Python directly (symlinks don't work in the sandbox):
```bash
/Users/benjaminfeuer/miniconda3/envs/otagent/bin/python your_script.py
```

**Syntax Checking**: Use the IDE MCP tool `mcp__ide__getDiagnostics` for checking Python syntax errors and linting issues. Do NOT use bash commands like `python -m py_compile` or `flake8` as the bash environment may have issues with output capture.

```
# Example: Check a file for errors
mcp__ide__getDiagnostics(uri="file:///path/to/file.py")
```

## Local Companion Codebases

When making changes to Harbor or SkyRL, edit the local repos and sync via git (commit, push, then pull on the cluster). Do NOT manually patch files on remote clusters.

All three codebases (Harbor, SkyRL, OT-Agent) are installed as **editable installs** (`pip install -e .`) on all clusters. After `git pull`, the updated source is immediately active — no reinstall needed.

- **Harbor**: `/Users/benjaminfeuer/Documents/harbor` — agent framework, environment backends, terminus agent
- **SkyRL**: `/Users/benjaminfeuer/Documents/SkyRL` — RL training framework, trainer, terminal_bench generator

**Jupiter conda environments**: Use `otagent` for all OT-Agent work (job launching, scripts, uploads). Use `curator` only for curator data-generation jobs.

## Repository Overview

ot-agent is a distributed training and evaluation system for large language models on HPC clusters. It consists of four main subsystems:

1. **Data Generation**: Task and trace generation pipelines using Harbor/Daytona
2. **SFT Training (DCFT)**: Supervised fine-tuning using LLaMA-Factory
3. **RL Training**: Reinforcement learning training with SkyRL (using GRPO algorithm)
4. **Evaluation**: Terminal-bench based evaluation system

## Architecture

### Directory Structure

- **`data/`**: Data generation pipelines - each subdirectory is a named pipeline
- **`hpc/`**: DCFT SFT training launcher (uses LLaMA-Factory)
- **`train/hpc/`**: OT-Agent RL training launcher (uses SkyRL framework)
- **`eval/`**: Evaluation systems for both TACC and JSC clusters
- **`database/unified_db/`**: Supabase registry for datasets, models, and agents
- **`scripts/`**: Utility scripts for database, datagen, harbor, vllm, etc.

Both HPC launchers share similar architecture:
- `launch.py`: Main entry point for job submission
- `hpc.py`: Cluster detection and configuration (Pydantic models)
- `arguments.py`: CLI argument parsing
- `sbatch/`: SLURM job templates (Jinja2 for RL, plain for SFT)
- `dotenv/`: Environment variable files per cluster
- `scripts/common.sh`: Shared bash utilities and aliases

### Key Distinction: Internet Access

The codebase handles two types of HPC clusters:

**Internet-enabled clusters** (TACC: Vista, Lonestar; ZIH: Capella, Alpha):
- Compute nodes can directly access HuggingFace Hub
- Standard dataset/model loading works

**No-internet clusters** (JSC: Jureca, Jupiter, Juwels; Leonardo):
- Compute nodes have NO internet access
- `pre_download_dataset()` function pre-downloads datasets/models on login nodes
- Downloads stored in `HF_HUB_CACHE` before job submission
- Training uses SSH tunnels and Ray for coordination

### Supported HPC Clusters

**TACC (Texas Advanced Computing Center)**:
- Vista: GH200 96GB GPUs, 552 nodes, internet access
- Lonestar (ls6): A100 40GB GPUs, 73 nodes, internet access

**JSC (Jülich Supercomputing Centre)**:
- Jureca: H100 94GB GPUs, 16 nodes, no internet
- Jupiter: GH200 96GB GPUs, 48 nodes, no internet
- Juwels: A100 40GB GPUs, 936 nodes, no internet

**ZIH (TU Dresden)**:
- Capella: H100 94GB GPUs, 146 nodes, internet access
- Alpha: A100 40GB GPUs, 37 nodes, internet access

**Leonardo** (CINECA):
- A100 64GB GPUs, 3456 nodes, no internet

### Data Generation System

`data/` contains named pipeline directories. Two approaches are supported:

**Declarative scripts (`generate.py`)**: Self-contained Python scripts for local/one-off runs
```bash
python data/<dataset>/generate.py --optional-flags
```

**Class-based generators (`generate_abstract.py`)**: Subclass `BaseDataGenerator` for HPC runs with managed vLLM endpoints
```bash
python -m hpc.launch \
  --job_type datagen \
  --datagen_script data/<dataset>/generate_abstract.py \
  --datagen_target_repo <org/repo> \
  --datagen_extra_args "--stage both --limit 2000"
```

Key modules in `data/generation/`: `base.py` (BaseDataGenerator), `schemas.py` (GenerationRequest/Result), `engines.py` (InferenceEngine implementations for OpenAI/Anthropic/vLLM)

**Curator sharded datagen (`run_curator_datagen_sharded.sbatch`)**: Multi-node data-parallel generation using vLLM + async_datagen.py. Default: 32 nodes (one vLLM server per node). Supports auto-resume via stable shard output dirs. Uses `--account=reformo` on Jupiter (not the default `jureap59` account).
```bash
# Submit with restart chain (recommended for long datasets):
FIRST=$(sbatch data/sbatches/run_curator_datagen_sharded.sbatch \
  <model> <input_dataset> <output_repo> [limit] [save_every] | awk '{print $4}')
PREV=$FIRST; for i in $(seq 1 6); do

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-thoughts/OpenThoughts-Agent](https://github.com/open-thoughts/OpenThoughts-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

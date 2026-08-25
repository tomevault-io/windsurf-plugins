---
trigger: always_on
description: > **TAP-Detect**: Temporal-Aware Perplexity Detection for AI-Generated Text
---

# AGENTS.md - TAP-Detect Implementation Guide

> **TAP-Detect**: Temporal-Aware Perplexity Detection for AI-Generated Text
> 
> This document provides complete developer instructions, scaffolding, and code-interface
> contracts for implementing TAP-Detect as specified in the project requirements.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Quick Start](#quick-start)
3. [Architecture](#architecture)
4. [Interface Contracts](#interface-contracts)
5. [Implementation Guide](#implementation-guide)
6. [Person A Tasks (Data Pipeline)](#person-a-tasks-data-pipeline)
7. [Person B Tasks (Model & Training)](#person-b-tasks-model--training)
8. [Review Process](#review-process)
9. [HPC Deployment](#hpc-deployment)
10. [Appendix: Research Paper Summaries](#appendix-research-paper-summaries)

---

## Project Overview

### Goal
Implement **TAP-Detect**, a novel AI-generated text detection method that addresses five critical 
limitations of traditional perplexity-based approaches by exploiting temporal dynamics in AI text 
generation.

### Key Innovation
AI models become increasingly predictable as they generate text (24-32% volatility reduction), 
while human writing maintains consistent unpredictability throughout.

### Target Performance
- **Accuracy**: 88-91%
- **Speed**: 10x faster than baseline methods
- **Minimum text length**: 150 tokens (vs 400-500 for competitors)

### Environment
- **Python**: 3.10.9
- **PyTorch**: 2.1.2+cu121
- **CUDA**: Driver 12.8, Toolkit 12.1
- **HPC**: Multi-node/multi-GPU with shared parallel filesystem

---

## Quick Start

### 1. Environment Setup

```bash
# Windows
scripts\setup_env.bat

# Linux/macOS
chmod +x scripts/setup_env.sh
./scripts/setup_env.sh

# Or manually with conda
conda env create -f environment.yml
conda activate tap-detect
```

### 2. Verify Installation

```bash
python scripts/verify_cuda.py
```

### 3. Run Tests

```bash
pytest tests/ -v
```

### 4. Train Model

```bash
# Single GPU
python training/train.py --config configs/default.yaml

# Multi-GPU (DDP)
./scripts/launch_torchrun.sh --config configs/default.yaml --gpus 4

# HPC (SLURM)
sbatch scripts/slurm_template.sh
```

---

## Architecture

### Eight Core Components

TAP-Detect implements eight interconnected components:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TAP-Detect Pipeline                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │  1. Key Token    │───▶│  2. Temporal     │───▶│  3. Uncertainty  │  │
│  │  Identification  │    │  Dynamics        │    │  Map Generation  │  │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘  │
│          │                       │                       │              │
│          ▼                       ▼                       ▼              │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │  4. Adaptive     │◀───│  5. Per-Window   │◀───│  6. Triple-      │  │
│  │  Sliding Windows │    │  Perplexity      │    │  Weighted Agg    │  │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘  │
│          │                                               │              │
│          ▼                                               ▼              │
│  ┌──────────────────┐                         ┌──────────────────┐     │
│  │  7. Final Score  │◀────────────────────────│  8. Threshold    │     │
│  │  Calculation     │                         │  Classification  │     │
│  └──────────────────┘                         └──────────────────┘     │
│                                                         │              │
│                                                         ▼              │
│                                            ┌──────────────────────┐    │
│                                            │ Output: AI/Human/    │    │
│                                            │ Uncertain + Score    │    │
│                                            └──────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| # | Component | Purpose | Key Algorithm |
|---|-----------|---------|---------------|
| 1 | Key Token ID | Select informative tokens | `top_30%(|PPL_long - PPL_short| × attention)` |
| 2 | Temporal Dynamics | Extract volatility features | Derivative dispersion, local volatility, decay ratio |
| 3 | Uncertainty Map | Measure token difficulty | `uncertainty[i] = 1 - max(P(token[i]))` |
| 4 | Adaptive Windows | Variable window sizing | Size based on local uncertainty |
| 5 | Per-Window PPL | Compute perplexity | Standard perplexity per window |
| 6 | Triple-Weighted | Aggregate with weights | Importance × Position × Uncertainty (learned) |
| 7 | Final Score | Combine signals | `w_ppl × weighted_ppl + w_temp × temporal` (learned) |
| 8 | Classification | Output decision | Thresholds: 0.3 (AI), 0.7 (Human) |

> **Note on Learnable Weights:** All combination weights (Components 2, 6, 7) are learned 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k25kar/TAP-Detect](https://github.com/k25kar/TAP-Detect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->

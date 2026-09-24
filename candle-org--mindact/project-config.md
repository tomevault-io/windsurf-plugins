---
trigger: always_on
description: MindAct is a PyTorch and Hugging Face native training and evaluation toolkit for embodied policies. It provides reproducible experiment tracking for imitation learning on desktop manipulation tasks, integrating LeRobot datasets/policies with LIBERO simulation benchmarks.
---

# MindAct Project - Claude Code Configuration

## Project Overview

MindAct is a PyTorch and Hugging Face native training and evaluation toolkit for embodied policies. It provides reproducible experiment tracking for imitation learning on desktop manipulation tasks, integrating LeRobot datasets/policies with LIBERO simulation benchmarks.

## Directory Structure

```
mindnlp/  (repository root)
├── .claude/
│   ├── settings.json
│   └── projects/
├── src/
│   └── mindact/
│       ├── cli/              # Command-line interface
│       ├── configs/          # Configuration schemas
│       ├── datasets/         # Dataset adapter protocols
│       ├── envs/             # Environment adapter protocols
│       ├── evaluation/       # Evaluation protocols
│       ├── experiments/      # Manifest and artifact management
│       ├── integrations/     # Lazy loading wrappers for lerobot/libero
│       ├── policies/         # Policy adapter protocols
│       ├── training/         # Training protocols
│       └── utils/            # Optional import handling
├── configs/
│   ├── experiments/          # YAML experiment configurations
│   └── README.md
├── examples/
│   └── libero/               # LIBERO integration examples
├── tests/
│   ├── unit/                 # Unit tests for core modules
│   └── integration/          # Integration tests with optional deps
├── docs/
│   ├── getting-started/
│   ├── concepts/
│   ├── guides/
│   └── contributing.md
├── benchmarks/               # Performance benchmarks
├── pyproject.toml            # PEP 621 package metadata
├── README.md
├── CLAUDE.md                 # This file
├── LICENSE
└── NOTICE
```

## Current Status (as of 2026-08-31)

**Version**: 0.1.0 (skeleton release)  
**Branch**: feature/mindact-v0.1 (active development)  
**Legacy branch**: legacy (preserves old MindNLP/MindTorch code)

### Implemented

- Protocol-based adapter interfaces (datasets, policies, environments, trainers, evaluators)
- Typed configuration system with YAML serialization
- Experiment provenance tracking (manifests + artifacts)
- Optional dependency handling (torch, lerobot, libero as extras)
- CLI skeleton with config validation
- Example YAML configurations
- Unit tests and skip-clean optional integration smoke tests
- MkDocs documentation skeleton and contributor guide
- Python 3.12/3.13 CI for tests, linting, and package builds

### Not Yet Implemented

- Training loop (Trainer protocol implementation)
- Evaluation loop (Evaluator protocol implementation)
- LeRobot dataset adapter
- LeRobot policy adapter
- LIBERO environment adapter
- Checkpoint loading/saving
- Metrics logging

---

## Core Design Principles

### 1. Protocol-Based Architecture

**CRITICAL**: All integration boundaries use `@runtime_checkable Protocol` from `typing`, not ABC or concrete base classes.

- **Why**: Allows external libraries to satisfy MindAct contracts without subclassing
- **Example**: LeRobot policies can be wrapped as `PolicyAdapter` without modification
- **Rule**: Never require inheritance from MindAct base classes

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class PolicyAdapter(Protocol):
    """Contract for policy implementations."""
    name: str
    revision: str | None
    
    def predict(self, observation: dict[str, Any]) -> dict[str, Any]: ...
    def load(self, checkpoint: str | Path) -> None: ...
```

### 2. Optional Dependencies

**CRITICAL**: MindAct core has ZERO required dependencies except numpy and pyyaml. All ML frameworks are optional extras.

- **Never** import `torch`, `lerobot`, or `libero` at module level
- **Always** use `require_module()` from `mindact.utils.imports` at runtime
- **Rationale**: Users can install only what they need; config validation works without PyTorch

```python
# BAD: Eager import
import torch
from lerobot import Dataset

# GOOD: Lazy loading
def load_lerobot_dataset(config: DatasetConfig):
    lerobot = require_module("lerobot", feature="LeRobot dataset loading")
    return lerobot.Dataset(config.source, split=config.split)
```

### 3. Frozen Dataclasses for All Configurations

**CRITICAL**: All configuration and result objects are frozen dataclasses. No mutable state.

- **Why**: Prevents accidental modification after validation; enables safe caching
- **Rule**: Use `@dataclass(frozen=True, kw_only=True)` for all configs and results
- **Exception**: None

```python
from dataclasses import dataclass

@dataclass(frozen=True, kw_only=True)
class TrainingConfig:
    """Training hyperparameters."""
    steps: int
    batch_size: int
    learning_rate: float
    checkpoint_every: int
```

### 4. Experiment Provenance

**CRITICAL**: Every training run generates a manifest tracking exact dataset/policy/checkpoint/evaluation lineage.

- **Manifest location**: `experiments/<run-id>/manifest.json`
- **Required fields**: experiment name, seed, dataset (source, revision, split), policy (architecture, checkpoint), training config, evaluation config, run_id, timestamp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [candle-org/MindAct](https://github.com/candle-org/MindAct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

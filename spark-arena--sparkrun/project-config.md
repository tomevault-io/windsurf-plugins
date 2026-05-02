---
trigger: always_on
description: provides solo-mode orchestration; runtimes override `run()`/`stop()`/`follow_logs()` for multi-node support.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**sparkrun** is a CLI tool for launching, managing, and stopping Docker-based LLM inference workloads on NVIDIA DGX
Spark systems. It orchestrates containers over SSH — no Slurm or Kubernetes required. The control machine doesn't need
to be a cluster member; it coordinates DGX Sparks remotely.

Each DGX Spark has one GPU with 128 GB unified memory, so tensor parallelism maps directly to node count (`--tp 2` = 2
hosts).

## Common Commands

```bash
# Install in development mode (editable)
uv sync

# Run full test suite
.venv/bin/python -m pytest tests/ -v

# Run a single test file
.venv/bin/python -m pytest tests/test_recipe.py -v

# Run a specific test
.venv/bin/python -m pytest tests/test_cli.py::test_run_command_basic -v

# Run with coverage
.venv/bin/python -m pytest tests/ --cov=sparkrun --cov-report=term-missing

# Lint (ruff, line-length 140, target py312)
ruff check src/ tests/
ruff format src/ tests/

# Run the CLI directly during development
.venv/bin/sparkrun --help
.venv/bin/sparkrun run --dry-run qwen3-1.7b-vllm

# Sync versions across packages (pyproject.toml + sparkrun-cc-plugin)
python scripts/update-versions.py
python scripts/update-versions.py --check   # CI-friendly verify
```

Versions are tracked in `versions.yaml` at the repo root and synced to all package files via
`scripts/update-versions.py`.

## Architecture

### Source Layout

```
src/sparkrun/
├── cli/                # Click CLI package (see CLI Architecture below)
├── core/               # Core data models, bootstrap, and business logic (see below)
├── runtimes/           # Runtime plugins (see below)
├── orchestration/      # SSH, Docker, InfiniBand, script execution primitives
├── models/             # HuggingFace model download, distribution, and VRAM estimation
├── containers/         # Container image distribution (docker save/load over SSH)
├── tuning/             # Triton fused MoE kernel tuning for SGLang and vLLM
├── builders/           # Container image builder plugins (docker-pull, eugr)
├── diagnostics/        # Host and run diagnostic collection (NDJSON output)
├── proxy/              # LiteLLM-based inference proxy gateway
├── benchmarking/       # Benchmark framework plugins and result export (llama-benchy)
├── utils/              # Shared helpers (coerce_value, suppress_noisy_loggers, etc.)
└── scripts/            # Embedded bash scripts (IB detection, container launch, etc.)
```

### Core Data Models (`core/`)

Core domain logic extracted from the top-level package. All imports use `sparkrun.core.*` (e.g.,
`from sparkrun.core.config import SparkrunConfig`).

| Module                  | Purpose                                                                              |
|-------------------------|--------------------------------------------------------------------------------------|
| `bootstrap.py`          | SAF plugin initialization, runtime and benchmarking framework discovery              |
| `config.py`             | `SparkrunConfig` — reads `~/.config/sparkrun/config.yaml`, cache dir resolution      |
| `registry.py`           | `RegistryManager` — git-based recipe registry system (see Registry System below)     |
| `recipe.py`             | `Recipe` loading, validation, v1→v2 migration, config chain via SAF Variables         |
| `cluster_manager.py`    | `ClusterManager` — named cluster CRUD (YAML files in `~/.config/sparkrun/clusters/`) |
| `hosts.py`              | Host resolution priority chain (CLI → file → cluster → default)                      |
| `pending_ops.py`        | PID-based lock files for in-progress operations                                      |
| `benchmark_profiles.py` | Benchmark profile discovery, resolution, and rendering across registries             |

### CLI Architecture (`cli/`)

The CLI was split from a single `cli.py` into a package for maintainability. The `__init__.py` defines the top-level
`main` Click group, registers all subcommands, and provides top-level aliases (`list`, `show`, `search`, `status`).

| Module            | Purpose                                                                                                                                                                                                                                                           |
|-------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `__init__.py`     | `main` Click group, command registration, top-level aliases                                                                                                                                                                                                       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spark-arena/sparkrun](https://github.com/spark-arena/sparkrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

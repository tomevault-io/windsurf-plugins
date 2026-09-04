---
trigger: always_on
description: This guide documents how to work effectively in the **parakeet-rocm** codebase. It covers setup, architecture, and the required coding rules enforced by Ruff + Pytest. When in doubt, prefer **correctness → clarity → consistency → brevity** (in that order).
---

# AGENTS.md — Parakeet-ROCm Agent Guide

This guide documents how to work effectively in the **parakeet-rocm** codebase. It covers setup, architecture, and the required coding rules enforced by Ruff + Pytest. When in doubt, prefer **correctness → clarity → consistency → brevity** (in that order).

## Table of Contents

- [Setup & Commands](#setup--commands)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Architecture & Patterns](#architecture--patterns)
- [Code Style & Patterns](#code-style--patterns)
  - [1) Correctness (Ruff F - Pyflakes)](#1-correctness-ruff-f---pyflakes)
  - [2) PEP 8 surface rules (Ruff E, W - pycodestyle)](#2-pep-8-surface-rules-ruff-e-w---pycodestyle)
  - [3) Naming conventions (Ruff N - pep8-naming)](#3-naming-conventions-ruff-n---pep8-naming)
  - [4) Imports: order & style (Ruff I - isort rules)](#4-imports-order--style-ruff-i---isort-rules)
  - [5) Docstrings — content & style (Ruff D + DOC)](#5-docstrings--content--style-ruff-d--doc)
  - [6) Import hygiene (Ruff TID - flake8-tidy-imports)](#6-import-hygiene-ruff-tid---flake8-tidy-imports)
  - [7) Modern Python upgrades (Ruff UP - pyupgrade)](#7-modern-python-upgrades-ruff-up---pyupgrade)
  - [8) Future annotations (Ruff FA - flake8-future-annotations)](#8-future-annotations-ruff-fa---flake8-future-annotations)
  - [9) Local ignores (only when justified)](#9-local-ignores-only-when-justified)
  - [10) Tests & examples (Pytest + Coverage)](#10-tests--examples-pytest--coverage)
  - [11) Commit discipline](#11-commit-discipline)
  - [12) Quick DO / DON’T](#12-quick-do--dont)
  - [13) Pre-commit (recommended)](#13-pre-commit-recommended)
  - [14) CI expectations](#14-ci-expectations)
  - [15) SOLID design principles — Explanation & Integration](#15-solid-design-principles--explanation--integration)
  - [16) Configuration management — environment variables & constants](#16-configuration-management--environment-variables--constants)
  - [Final note (code style)](#final-note-code-style)
- [Git / PR Workflow](#git--pr-workflow)
- [Boundaries](#boundaries)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)

______________________________________________________________________

## Setup & Commands

### Install

**Recommended (Docker / ROCm-enabled runtime):**

```bash
pip install pdm
pdm install -G rocm,webui
docker compose build
```

**Local development prerequisites:** Python 3.10, ROCm 7.0, PDM >= 2.15, and ROCm PyTorch wheels configured in your PDM source list. See `README.md` for the full setup steps and fallbacks.

If you are not using Docker, the fallback dependency install used in README is:

```bash
pdm install -G rocm,webui
# or run in a virtualenv
pip install -r requirements-all.txt
```

### Environment configuration

```bash
cp .env.example .env
```

Environment variables are loaded once in `parakeet_rocm/utils/constant.py`. Do not access `os.environ` elsewhere.

### Run / Dev

```bash
parakeet-rocm --help
parakeet-rocm transcribe data/samples/sample.wav
docker compose up
```

### Tests

```bash
# Fast unit tests only
pdm run pytest tests/unit/

# Full suite (unit + integration + e2e)
pdm run pytest

# Coverage
pdm run pytest tests/unit/ --cov=parakeet_rocm --cov-report=term-missing:skip-covered
```

### Lint / Format

```bash
pdm run ruff check --fix .
pdm run ruff format .
```

### Build

```bash
pdm build
```

### Tooling & scripts

```bash
pdm run local-ci
pdm run srt-diff-report  # PDM dev script (not an installed console command)
bash scripts/clean_codebase.sh
```

______________________________________________________________________

## Project Structure

```text
parakeet_rocm/
├── parakeet_rocm/           # Core package (CLI, pipeline, WebUI, utils)
├── tests/                   # Unit, integration, e2e, and slow tests
├── scripts/                 # CI helpers, report generators, local tooling
├── docs/                    # Additional docs (if present)
├── data/                    # Sample input data
├── output/                  # Default output location
├── docker-compose.yaml      # Docker entrypoints
├── Dockerfile               # Container build
├── project-overview.md      # Architecture + patterns reference
├── TESTING.md               # Test strategy + markers
├── .env.example             # Environment variable reference
└── pyproject.toml           # Dependencies, scripts, Ruff, Pytest config
```

**Key package areas:**

- `parakeet_rocm/cli.py`: Typer CLI entrypoint and command wiring.
- `parakeet_rocm/transcription/`: File processing pipeline and protocols.
- `parakeet_rocm/chunking/`: Chunking and merge strategies.
- `parakeet_rocm/formatting/`: Output formatter registry and implementations.
- `parakeet_rocm/webui/`: Gradio WebUI app and UI components.
- `parakeet_rocm/utils/`: Constants, env loader, path helpers, audio I/O.

**Path aliases:** None. Use absolute imports from `parakeet_rocm`.

______________________________________________________________________

## Tech Stack

### Core

- Python 3.10 (runtime and typing target)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beecave-homelab/parakeet_rocm](https://github.com/beecave-homelab/parakeet_rocm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->

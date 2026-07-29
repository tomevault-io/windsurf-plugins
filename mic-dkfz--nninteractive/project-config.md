---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

nnInteractive is a 3D interactive medical image segmentation framework. It supports diverse prompt types (points, scribbles, bounding boxes, lasso) using 2D interactions to generate full 3D segmentations. This repository contains the **inference-only** package.

## Build & Development Commands

```bash
# Install from source (editable). This repo builds TWO distributions that share the
# `nnInteractive` namespace (see "Two-distribution layout" below): the torch-free client
# must be installed first because the full package depends on it.
pip install -e ./client   # nninteractive-client (torch-free wire client)
pip install -e .          # nnInteractive (full stack; depends on the client)

# Install with dev tools
pip install -e ".[dev]"

# Code formatting
black nnInteractive/

# Linting
ruff check nnInteractive/ --fix

# Spell checking
codespell --skip='.git,*.pdf,*.svg'

# Pre-commit hooks (after: pre-commit install)
pre-commit run --all-files
```

There is no test suite in this repository.

## Architecture

### Two-distribution layout (full package + lightweight client)

The repo builds **two pip distributions** that share the single `nnInteractive` import namespace:

- **`nninteractive-client`** — source under `client/`. Ships *only* `nnInteractive.inference.remote`
  (the torch-free remote client + the shared wire protocol/serialization). Depends on just
  `numpy`, `httpx`, `blosc2`. Built from `client/pyproject.toml`.
- **`nnInteractive`** (full) — source at the repo root. Ships everything else (local engine,
  server, model management, …) and **depends on `nninteractive-client`**. Built from the root
  `pyproject.toml`.

How the shared namespace works:
- `nnInteractive` and `nnInteractive.inference` are **PEP 420 namespace packages** — neither
  has an `__init__.py`; the two distributions populate disjoint files into the same directory.
  Both builds therefore use `namespaces = true`, and the full build `exclude`s
  `nnInteractive.inference.remote*` (client-owned) and `nnInteractive.supervoxel*` (separate
  package). **Do not add an `__init__.py` to either of those two dirs** — it would make the two
  distributions ship the same file and break the clean split.
- Because the layout is layered (disjoint files, full *depends on* client), the two coexist
  cleanly: `pip install nnInteractive` pulls the client; uninstalling the client leaves the full
  install intact; a client-only machine upgrades with `pip install nnInteractive` (no uninstall).
- There is **no `nnInteractive.__version__`** (namespace package has no module to carry it).
  Read the version via `importlib.metadata.version("nnInteractive")` /
  `version("nninteractive-client")`. `nnInteractiveInferenceSession.INFERENCE_SESSION_VERSION`
  does exactly this.
- Full-only imports from a client-only install raise a friendly "`pip install nnInteractive`"
  error via a last-resort `sys.meta_path` finder registered in
  `client/nnInteractive/inference/remote/_full_required.py` (installed when the remote client is
  imported). It is a no-op when the full package is present.
- The full server (`server/app.py`) imports the wire serialization from
  `nnInteractive.inference.remote.serialization`, which is now provided by the client dependency.

### Core Class: `nnInteractiveInferenceSession` (`nnInteractive/inference/inference_session.py`)

Session-based inference engine (~2000 lines) that manages state across multiple predictions. This is the main user-facing API.

**Workflow**: `initialize_from_trained_model_folder()` → `set_image()` → `set_target_buffer()` → `add_*_interaction()` (repeatable)

- **Background threading**: Image preprocessing and interaction initialization run in a ThreadPoolExecutor (max 2 workers) via futures
- **AutoZoom**: Adaptive patch selection with border change detection; zooms out up to 4x if predictions touch crop boundaries
- **Refinement**: After coarse prediction, difference maps identify regions needing fine-grained re-prediction
- **Memory management**: Selective CPU/GPU transfers, pre-allocated tensors, half-precision interactions, pinned memory (disabled on Linux kernel 6.11)

**Important API constraints**:
- `use_torch_compile` is supported. The session default is `False`, but the **server enables it by default** (disable with `--no-torch-compile`). When enabled, the first prediction is slow (lazy compilation on the first forward pass) but subsequent ones are faster; the one-time compile cost is amortized across the long-lived server process. `nnInteractiveInferenceSession.warmup()` runs a single dummy forward pass at the network's only input shape (`[1, num_input_channels + num_interaction_channels, *patch_size]` — every prediction path uses this shape) to trigger compilation up front; the server calls it at startup so clients never see the first-prediction delay
- `target_buffer` must be 3D (shape `[X, Y, Z]`), not 4D
- Scribble and lasso images must match `original_image_shape[1:]` (the original uncropped spatial shape)
- `add_initial_seg_interaction()` **resets all existing interactions** (see WARNING in its docstring)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MIC-DKFZ/nnInteractive](https://github.com/MIC-DKFZ/nnInteractive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

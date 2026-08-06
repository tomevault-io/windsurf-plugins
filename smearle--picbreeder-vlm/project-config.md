---
trigger: always_on
description: The venv you need is here: `.venv/bin/python`
---

The venv you need is here: `.venv/bin/python`

# Picbreeder-VLM Codebase Documentation

This document serves as a guide for agents and developers working on the Picbreeder-VLM project. It outlines the project architecture, key files, and operational workflows.

## 📦 Package layout

The Python source lives in the **`picbreeder_vlm/`** package (install once with
`pip install -e .`). Modules are grouped into subpackages:

| subpackage | what's in it |
| --- | --- |
| `picbreeder_vlm.core` | config, constants, utils, rendering, `neat_components`, `picbreeder_reproduction`, `archive_manager`, `genome_json`, artifacts |
| `picbreeder_vlm.vlm` | `vlm_backends`, chat, prompts, im_query, personalities, model_loader |
| `picbreeder_vlm.agents` | `collaborative_multi_agent`, `agent_runner` |
| `picbreeder_vlm.experiments` | `sweep`, `sweep_configs`, sweep utils, `experiment_cli` |
| `picbreeder_vlm.analysis` | coverage / embedding / captioning / phylogeny / ratings metrics |
| `picbreeder_vlm.viz` | `visualize_*`, `render_*` figure generators |
| `picbreeder_vlm.niches` | `clip_noun_niche_*` CLIP evolution-strategy experiments |
| `picbreeder_vlm.nouns` | noun-list / ImageNet vocabulary wrangling |
| `picbreeder_vlm.bench` | VLM benchmarking, probing, ad-hoc tests |

Run modules as `.venv/bin/python -m picbreeder_vlm.<sub>.<module>` (e.g.
`... -m picbreeder_vlm.experiments.sweep`). The bare filenames below name the
module; e.g. **`sweep.py`** ⇒ `picbreeder_vlm.experiments.sweep`.

> **Pickle compat:** thin shims at the repo root (`neat_components.py`,
> `config.py`, `picbreeder_reproduction.py`, `archive_manager.py`, `rendering.py`)
> re-export from the package so existing archive/HF genome `.pkl` files (which
> store the original module paths) still `pickle.load`. Don't delete them.

## ✅ Tests

```bash
uv pip install -r requirements-test.txt   # NOT requirements.txt: no torch / vLLM
pytest                                    # ~60 tests, a couple of seconds
```

`.github/workflows/tests.yml` runs this on every push and PR (Python 3.11 + 3.13).
The suite exists to catch what a reorg breaks quietly, so it pins the contracts
rather than the implementation:

*   **Run-directory names** (`tests/test_config.py`) are goldens copied from published
    runs. `ensure_valid_config` builds them, sweeps and every tool glob them, and the
    HF archive is keyed by them. A rename orphans data — if a test here fails, do not
    "update the golden" without knowing what it costs.
*   **Pickle shims** (`tests/test_compat_shims.py`) must resolve to the *same class
    objects* as the package, or archived genomes stop unpickling.
*   **The NEAT preset** (`tests/test_neat_preset.py`) has to load and render
    deterministically; archive images are regenerated from genomes on demand.
*   **`core.neat_components` must stay importable without torch / vLLM / google-genai.**
    That is enforced by a test, and it is why `build_neat_config` lives there rather
    than in `agents.collaborative_multi_agent`.

Keep tests import-light: `picbreeder_vlm.analysis.*` pulls in torch and open_clip at
module scope, so `tests/test_analysis_naming.py` inspects those modules with `ast`
instead of importing them.

## 🗺️ Project Roadmap

The codebase is organized into several distinct layers, from high-level orchestration to low-level evolutionary mechanics.

### 1. Orchestration & Configuration
*   **`sweep.py`**: The main entry point. Orchestrates experiments (sweeps) locally or on Slurm. It generates individual run configurations and launches them.
*   **`config.py`**: Defines the `PicbreederConfig` dataclass. This is the single source of truth for experiment settings (grid size, model choice, evolution parameters).
*   **`sweep_configs.py`**: Defines the search spaces for hyperparameters. Contains `SweepConfig` and named sweep classes (e.g., `ChatHistoryTurnsSweep`).

### 2. Core Agent Logic
*   **`collaborative_multi_agent.py`**: The heart of the simulation. It runs the main loop where agents join, evolve images, and publish to the shared archive.
*   **`agent_runner.py`**: Manages the lifecycle of a single agent process, handling its interaction with the NEAT population and the VLM.

### 3. VLM & AI Layer
*   **`vlm_backends.py`**: The unified interface for AI models. Abstracts away differences between Gemini, Qwen, and other models. Handles image/text queries and chat history.
*   **`chat.py`**: Manages conversation history and context for agents.
*   **`prompts.py`**: Stores system prompts and goal definitions used to guide the VLMs.

### 4. Evolutionary Engine (NEAT)
*   **`picbreeder_reproduction.py`**: Custom NEAT reproduction logic. Implements the specific crossover and mutation rules that emulate the original Picbreeder web app.
*   **`neat_components.py`**: Core NEAT utilities, including the `PicbreederGenome` class and stagnation logic.
*   **`interactive_config_color`**: The NEAT preset every entry point evolves against.
    It sits beside `picture2d.py`, the renderer that consumes it, because nothing ever
    swaps it out — it is effectively source. Resolve it via
    `picbreeder_vlm._paths.NEAT_CONFIG_PATH`, never by hardcoding the path.

### 5. Archive & State Management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smearle/picbreeder-vlm](https://github.com/smearle/picbreeder-vlm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

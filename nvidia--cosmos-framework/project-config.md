---
trigger: always_on
description: Read this file first — it is the canonical map for navigating the Cosmos repository and stays up to date.
---

# AGENTS.md — Cosmos-Framework

Read this file first — it is the canonical map for navigating the Cosmos repository and stays up to date.

**Cosmos** is a framework for training and serving world foundation models. Everything lives in a single top-level `cosmos_framework/` Python package:

- **Training infrastructure** — top-level subpackages under `cosmos_framework/` (data, model, trainer, callbacks, checkpoint, …).
- **Inference infrastructure** — `cosmos_framework/inference/` (Diffusers / Transformers / vLLM-friendly inference core, online serving via Ray + Gradio).
- **Backend packages** — `packages/{diffusers,transformers,vllm}-cosmos3/` provide library-style shims that load Cosmos3 checkpoints into the respective ecosystems.
- **Entry-point scripts** — `cosmos_framework/scripts/` (`train.py`, `inference.py`, `eval.py`, `export_model.py`, …) invoked as `python -m cosmos_framework.scripts.<name>`. Primary training entry point: `cosmos_framework.scripts.train` driven by a structured, pydantic-validated TOML interface (`--sft-toml=<recipe-toml>`); the schema lives at [`cosmos_framework/configs/toml_config/sft_config.py`](./cosmos_framework/configs/toml_config/sft_config.py) and the canonical recipe pattern is documented in [`examples/README.md`](./examples/README.md).

> All paths below are relative to the repository root (the directory containing `pyproject.toml`, the `cosmos_framework/` Python package, and `packages/`).

## Commands

| Task                   | Command                                             |
| ---------------------- | --------------------------------------------------- |
| Lint                   | `uv run ruff check .`                               |
| Format check           | `uv run ruff format --check .`                      |
| Auto-fix lint + format | `uv run ruff check --fix . && uv run ruff format .` |
| Type-check             | `uv run pyrefly check`                              |
| Test (all)             | `uv run pytest`                                     |
| Test (single file)     | `uv run pytest --capture=no <path>`                 |

Config files: `.ruff.toml` (ruff), `pyrefly.toml` (pyrefly), `.pytest.toml` (pytest), `conftest.py` (pytest fixtures).

A `justfile` is provided at the root with longer recipes (`just install`, `just lint`, `just test`, `just docker-cu130`).

## Rules

- Always answer questions with references to code or documentation in `file:line` format.
- When unsure, point the user to the closest doc rather than guessing.
- Keep this file short. Link out to skills and docs for detail — this file is included in every prompt.
- Inference code belongs under `cosmos_framework/inference/`; training infrastructure belongs under the other `cosmos_framework/` subpackages. Don't blur the two — if you find yourself adding training-time imports inside `cosmos_framework/inference/` (or vice versa), reconsider.

## Key File Locations

### Training (`cosmos_framework/`)

| What                                                 | Where                                           |
| ---------------------------------------------------- | ----------------------------------------------- |
| Algorithms (losses, RL, reward)                      | `cosmos_framework/algorithm/{loss,reward,rl}`   |
| Training loop                                        | `cosmos_framework/trainer/`                     |
| Models + parallelism                                 | `cosmos_framework/model/`                       |
| Datasets / data loading                              | `cosmos_framework/data/`                        |
| Checkpoint I/O                                       | `cosmos_framework/checkpoint/`                  |
| Callbacks (logging, eval)                            | `cosmos_framework/callbacks/`                   |
| RL workers (rollout, reward, reference, simulations) | `cosmos_framework/workers/`                     |
| Controller / orchestrator                            | `cosmos_framework/controller/`                  |
| Launchers (Slurm, torchrun, k8s)                     | `cosmos_framework/launcher/`                    |
| Evaluation harness                                   | `cosmos_framework/evaluation/`                  |
| CLI tools                                            | `cosmos_framework/tools/`, `tools/` (repo root) |

For a per-subpackage tour with descriptions, see [`docs/code_structure.md`](./docs/code_structure.md).

### Inference (`cosmos_framework/inference/`)

| What                     | Where                                                                            |
| ------------------------ | -------------------------------------------------------------------------------- |
| CLI entry point          | `cosmos_framework/scripts/inference.py`                                          |
| Args / param definitions | `cosmos_framework/inference/args.py`                                             |
| Per-modality defaults    | `cosmos_framework/inference/defaults/<mode>/sample_args.json`                    |
| Model / inference core   | `cosmos_framework/inference/model.py`, `cosmos_framework/inference/inference.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/cosmos-framework](https://github.com/NVIDIA/cosmos-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

---
trigger: always_on
description: NeMo AutoModel is a PyTorch-native training framework for LLMs, VLMs, diffusion
---

# NeMo AutoModel -- Guide for AI Agents

NeMo AutoModel is a PyTorch-native training framework for LLMs, VLMs, diffusion
models, and retrieval models. It integrates with HuggingFace Transformers via
custom `NeMoAuto*` wrapper classes, uses YAML-driven recipe configs, and relies
on FSDP2/HSDP/DDP/DTensor/DeepEP for distributed training.

This document is the top-level reference for any AI agent working in this
repository. Read it first, then consult the relevant skill file for the task at
hand.

---

## Coding Style

- **Explicit over implicit.** Inline logic where possible; avoid hiding behavior
  behind unnecessary layers of indirection.
- **No speculative abstractions.** Do not add features, parameters, or
  generalization beyond what is explicitly asked for.
- **Formatter:** `ruff` with a line length of 120 and double quotes.
  Run `ruff format .` then `ruff check --fix .` before committing.
- **Type hints** are required on all public API signatures (functions, methods,
  class attributes exposed in `__init__.py`).
- **Docstrings** follow Google style.
- **Optional dependencies** must be guarded with `safe_import()` from
  `nemo_automodel.shared.import_utils`. Never let an optional import crash
  module loading.
- **Copyright header.** Every Python file must start with the NVIDIA copyright
  block. Do not remove or modify it.
- **Package management.** The project uses `uv`. Do not introduce `pip install`
  commands in scripts or docs, instead use `uv`.
- **Python version.** 3.10+ required. PyTorch 2.6+.

---

## Git & PR conventions

- **Branch names** use the format `<github-handle>/<type>/<short-desc>`
  (e.g. `jdoe/fix/rope-scaling`).
- **Commit messages** follow [Conventional Commits](https://www.conventionalcommits.org/):
  `type(scope)?: description` — e.g. `fix(ci): retry apt-get on mirror failures`.
- **PR titles** must match the same format. The CI `Validate PR title` check
  enforces this; a non-conforming title will fail the check.
  Valid types: `feat` `fix` `docs` `style` `refactor` `perf` `test` `build`
  `ci` `chore` `revert` `cp`. Title must be ≤ 80 characters.
- **Never** use bracket-prefixed styles such as `[ci] fix: …` — those will
  fail validation.

---

## Architecture Overview

```
automodel <command> <domain> -c <config.yaml>
    |
    v
_cli/app.py          -- routes command + domain to recipe scripts
    |
    v
recipes/             -- main training / eval entry points
  llm/
  vlm/
  diffusion/
  retrieval/
    |
    v
components/          -- modular building blocks
  models/            -- 27+ model families (LLM, VLM, MoE, ...)
  datasets/          -- LLM, VLM, diffusion data pipelines
  distributed/       -- FSDP2, HSDP, DDP utilities
  checkpoint/        -- async DCP, SafeTensors
  quantization/      -- FP8, QAT, calibration
  _peft/             -- LoRA, QLoRA adapters
  launcher/          -- Slurm, SkyPilot job submission
    |
    v
_transformers/       -- HuggingFace bridge
  auto_model.py      -- NeMoAutoModelForCausalLM, NeMoAutoModelForImageTextToText, ...
  registry.py        -- MODEL_ARCH_MAPPING (model registration)
  capabilities.py    -- per-model feature detection flags
  infrastructure.py  -- device mesh setup for distributed training

_diffusers/          -- diffusion pipeline wrapper
  NeMoAutoDiffusionPipeline
```

### Entry Point

`_cli/app.py` parses `automodel <command> <domain>` and dispatches to the
matching recipe script. The `-c` flag points to a YAML config that drives all
component construction.

### Recipes

Files under `recipes/` are the primary training entry points. Each recipe
assembles a model, optimizer, dataloader, and trainer from its YAML config,
then runs the training loop.

### Components

Everything under `components/` is a self-contained building block. Components
are composed by recipes, never by each other (no hidden cross-component
imports).

### Transformers Bridge

`_transformers/` is the integration layer with HuggingFace:

- `auto_model.py` -- defines the `NeMoAuto*` classes that wrap
  `PreTrainedModel` with NeMo-specific functionality (distributed init,
  checkpoint hooks, backend dispatch).
- `registry.py` -- `MODEL_ARCH_MAPPING` maps architecture strings to model
  classes. Every new model must be registered here.
- `capabilities.py` -- declares per-model feature flags (supports_fp8,
  supports_moe, has_combined_qkv, etc.). These flags drive conditional logic
  throughout the framework.
- `infrastructure.py` -- builds the device mesh for FSDP2/HSDP and manages
  process-group lifecycle.

### Diffusers Bridge

`_diffusers/` wraps HuggingFace diffusion pipelines via
`NeMoAutoDiffusionPipeline`, providing the same recipe-driven config and
distributed training interface used by LLM/VLM recipes.

---

## Model Conventions

### Directory Layout

Each model lives under `components/models/<name>/` and contains:

| File                    | Purpose                                           |
|-------------------------|---------------------------------------------------|
| `model.py`             | Model class (inherits `PreTrainedModel` + `HFCheckpointingMixin`) |
| `state_dict_adapter.py`| Weight key mapping between HF and NeMo formats    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-NeMo/Automodel](https://github.com/NVIDIA-NeMo/Automodel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

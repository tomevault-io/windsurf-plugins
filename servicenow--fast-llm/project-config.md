---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Installation

```bash
# Full install with GPU support (requires CUDA)
pip install -e ".[CORE,OPTIONAL,DEV]"

# CPU-only install (for IDE support, no GPU required)
FLASH_ATTENTION_SKIP_CUDA_BUILD=TRUE FLASH_ATTENTION_FORCE_BUILD=TRUE pip install -e ".[CORE,DEV]" --no-build-isolation
```

### Pre-commit hooks

```bash
pip install pre-commit
pre-commit install
```

Hooks run Black (line length 119), isort, autoflake, and pyupgrade automatically on commit.

### Running tests

Always redirect output to a file to avoid truncation, e.g. `pytest ... 2>&1 | tee /tmp/fast_llm_tests/pytest_out.txt`. Use `/tmp/fast_llm_tests/` as the output directory (create it if needed).

```bash
# All tests
pytest -v -n 6 tests/

# Single test file or function
pytest -v tests/layers/test_lm_losses.py
pytest -v tests/layers/test_lm_losses.py::test_name

# Run extra-slow tests (disabled by default)
pytest -v -n 6 --run-extra-slow tests/

# Filter by model type
pytest -v -n 6 --models gpt tests/

# Test Triton kernels on CPU (no GPU required)
TRITON_INTERPRET=1 pytest -v tests/layers/test_lm_losses.py
```

The test suite sets `FAST_LLM_SKIP_TRITON_AUTOTUNE=TRUE` automatically. Tests that require distributed execution spawn child processes via `torchrun`. `TRITON_INTERPRET=1` enables the Triton interpreter so Triton kernels run on CPU — use this when developing or debugging Triton code without a GPU.

When working with external models, run as a **separate** pytest invocation — combining `tests/` and `fast_llm_external_models/tests/` in one run causes OOM:

```bash
pytest -v -n 6 fast_llm_external_models/tests/
```

Tests in `tests/models/` chain via pytest-depends and must be run as a whole file. Use `--models <name>` to filter; never use `-k` or `::test_name`, which breaks the dependency chain (causes "dependency not found" failures).

### CLI

```bash
# General form
fast-llm <subcommand> [--config config.yaml] [key=value overrides...]

# Validate config without running
fast-llm train gpt --config config.yaml --validate

# Example: train GPT
fast-llm train gpt --config examples/mistral-4-node-benchmark.yaml
```

## Design principles

<!-- Sync with docs/contributing/contributing.md → ## 🧱 Design principles. Bullet titles and bodies must stay byte-identical (modulo prose-vs-list markdown punctuation: `**Title.**` here vs `**Title**:` in contributing.md). When you change one, update the other in the same commit. -->

- **Generalize rather than special-case.** New features should extend existing abstractions, not create parallel ones for a specific use case. If `Attention` doesn't cover a new model variant, extend its config rather than introducing `MyModelAttention`. Same principle for losses, MLP variants, normalization layers — prefer parameterizing the existing module over forking it.
- **No overhead when unused.** A new feature must add no measurable cost on the disabled path: no new kernel launches, GPU sync points, or slower GPU code paths; no CPU work added to training hot loops (forward/backward, schedule loop, per-step dataloader path); no cost that scales with model size, sequence length, batch size, or step count. Trivial additions outside hot loops — a config-flag branch, a one-shot validation in `__init__` — are fine. Gate new behavior behind a config flag that short-circuits cheaply when off.
- **No deadweight.** Don't add modules, classes, abstractions, or code paths that don't pull their weight, or config options that don't toggle meaningful behavior. If a new helper ends up with one caller, inline it; if a new branch has no real consumer, drop it. Three similar lines beats a premature abstraction.
- **Trust internal boundaries.** Validate at system boundaries (user input, external APIs, file formats); trust internal callers and framework invariants. Don't add `try/except`, input validation, fallbacks, or "can't happen" guards on code you control — let it crash so the bug surfaces clearly.

## Architecture

### Configuration system (`fast_llm/config.py`)

The core infrastructure. Every config is a frozen dataclass decorated with `@config_class()` that inherits from `Config`. Fields use `Field(default=..., desc=..., hint=FieldHint.X)` with hints that control serialization verbosity and validation:

- `FieldHint.architecture` — defines model structure; compared across checkpoints
- `FieldHint.core` — always required explicitly
- `FieldHint.optional/performance/stability/feature/expert` — optional tuning knobs
- `FieldHint.derived` — computed from other fields, never serialized

Dynamic dispatch (for YAML `type:` keys) uses `@config_class(dynamic_type={BaseClass: "name"})`. The registry enables subclass selection from config files.

`RunnableConfig` (in `fast_llm/engine/config_utils/runnable.py`) extends `Config` with CLI parsing. `fast-llm train gpt` chains two levels of dynamic type dispatch: `train` selects the trainer subcommand, `gpt` selects `GPTModelConfig`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ServiceNow/Fast-LLM](https://github.com/ServiceNow/Fast-LLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

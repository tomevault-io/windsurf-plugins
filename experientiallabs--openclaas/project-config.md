---
trigger: always_on
description: **Understand the root cause before writing any code.** When something breaks, trace the error back to its systemic origin — don't patch at the point where the symptom appears. Check logs, environment variables, config, and runtime state before assuming the code is wrong.
---

# CLaaS - Claude Code Guidelines

## Debugging Philosophy

**Understand the root cause before writing any code.** When something breaks, trace the error back to its systemic origin — don't patch at the point where the symptom appears. Check logs, environment variables, config, and runtime state before assuming the code is wrong.

**Never add local workarounds for systemic issues.** Do not add try/except fallbacks, filtering, isinstance guards, or other defensive code to mask an error you don't fully understand. These hacks erode the codebase over time and hide real bugs. If the fix feels like a bandaid, you haven't found the real problem yet.

**Treat errors as signals, not obstacles.** An unexpected value or failed assertion means something upstream is wrong. Trace the data flow end-to-end: where was this value produced? What configuration or state fed into it? The goal is durable, correct software — not silencing errors.

## Code Quality Rules

### After Every Change

Run lint and type checking after every code change:

```bash
uv sync --extra dev

# Lint and auto-fix
uv run ruff check claas/ tests/ --fix

# Type check (import errors for modal/torch/vllm are expected)
uv run ty check
```

Note: GPU dependencies (modal, torch, vllm, transformers, peft) are not installed locally. `ty check` will report `unresolved-import` errors for these - this is expected and can be ignored.

### Run Tests

```bash
uv run pytest tests/ -v -m "not integration"
```

## Project Structure

```text
claas/
├── __init__.py
├── api.py                               # FastAPI endpoints + inference proxy (entrypoint)
├── index.html                           # Dashboard template
│
├── core/                                # Shared types & config
│   ├── __init__.py
│   ├── config.py                        # Centralized env var config (get_config)
│   └── types.py                         # Pydantic models, TypedDicts (ChatMessage, etc.)
│
├── inference/                           # Inference backend abstraction
│   ├── __init__.py                      # Factory: get_inference_backend(kind)
│   ├── base.py                          # Abstract InferenceBackend + result dataclasses
│   ├── tinker.py                        # Tinker SDK implementation
│   ├── vllm.py                          # vLLM forwarding implementation
│   ├── cache.py                         # CompletionCache + CompletionCacheEntry
│   └── helpers.py                       # strip_thinking, extract_final_channel, SSE helpers
│
├── training/                            # Training pipeline
│   ├── __init__.py
│   ├── distillation.py                  # Shared SDPO trainer logic
│   ├── sdpo_loss.py                     # SDPO loss computation (core algorithm)
│   ├── storage.py                       # LoRA storage (Modal Volume or local fs)
│   ├── teacher_helpers.py               # Pure teacher prompt functions
│   └── engine/                          # Pluggable training backends
│       ├── __init__.py                  # get_training_engine() factory
│       ├── base.py                      # TrainingEngine abstract interface
│       ├── local/engine.py              # Local GPU execution
│       ├── modal/engine.py              # Modal remote execution
│       └── tinker/engine.py, state.py   # Tinker SDK execution
│
├── modal/                               # Modal deployment modules
│   ├── __init__.py
│   ├── deploy.py                        # Unified Modal app deployment
│   └── worker.py                        # Modal DistillWorker class
│
├── dashboard/                           # Web dashboards
│   ├── __init__.py
│   ├── pagination.py                    # Shared pagination helpers
│   ├── feedback_dashboard.html          # Feedback dashboard template
│   ├── eval_dashboard.html              # Eval dashboard template
│   └── eval_dashboard.py               # Eval results dashboard
│
├── eval/                                # Eval harness (Hydra config)
│   ├── __init__.py
│   ├── __main__.py                      # `python -m claas.eval` entry point
│   ├── config.py                        # Hydra config loading (load_config / build_harness_config)
│   ├── configs/
│   │   ├── base.yaml                    # Default Hydra YAML config
│   │   └── preference/                  # Per-preference YAML configs
│   │       ├── no_emoji.yaml
│   │       ├── concise.yaml
│   │       └── identity.yaml
│   ├── types.py                         # EvalConfig dataclass, metric types
│   ├── runner.py                        # Main eval loop (run_harness)
│   ├── preferences.py                   # YAML-based preference loader (hydra.utils.instantiate)
│   ├── plotting.py                      # Matplotlib plot generation
│   ├── metrics/                         # Measurement implementations
│   │   ├── __init__.py                  # Re-exports: Metric, build_metrics, etc.
│   │   ├── registry.py                  # Metric protocol + registry
│   │   ├── verifiers.py                 # Verifier protocol + callable verifier classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [experientiallabs/openclaas](https://github.com/experientiallabs/openclaas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

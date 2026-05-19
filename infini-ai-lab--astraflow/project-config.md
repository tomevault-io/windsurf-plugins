---
trigger: always_on
description: <!-- Operations guide for AI coding agents working on AstraFlow. -->
---

<!-- Operations guide for AI coding agents working on AstraFlow. -->

# AGENTS.md — AstraFlow Agent Operations Guide

## TL;DR for coding agents

- **Runtime**: Distributed GPU clusters (FSDP2 / Megatron). Assume
  containerized, multi-GPU, multi-node execution; do not invent
  standalone local runs.
- **Architecture**: Four cleanly separated components — Dataflow,
  Train Worker, RaaS, Workflow. See
  `docs/en/architecture/overview.md` for the long form.
- **Testing**: Most integration / FSDP / Megatron / RPC tests require
  multi-GPU hardware. Public CI (`.github/workflows/`) runs only lint,
  format, and docs — no GPU tests. State skips explicitly when you
  cannot run a suite.
- **Tooling**: `.pre-commit-config.yaml` runs Ruff (lint + format),
  mdformat, nbstripout, file-hygiene hooks, and the CLI doc generator.
  Install hooks with `pre-commit install` before submitting patches.
- **Formatting**: Ruff (`ruff==0.14.9`) is the single source of truth —
  both pre-commit and the `format-check` CI run `ruff check` and
  `ruff format`. The `[tool.black]` block in `pyproject.toml` is
  legacy and unused.
- **Docs**: Source lives under `docs/` (Sphinx). English pages
  under `docs/en/` with an `index.rst` toctree.
- **Collaboration**: Before non-trivial edits, outline the proposed
  plan and confirm with the user.

When unsure, leave a `TODO(agent)` comment and note the constraint in
your response.

## Repository map

- `astraflow/` — Top-level Python package, split by component:
  - `astraflow/dataflow/` — Async data flow, rollout buffering,
    replay, staleness management, HTTP service layer. Key classes:
    `AstraFlow` (`service.py`), `DataAcquisition`
    (`data_acquisition.py`), `DataServing` (`data_serving.py`),
    `RaaS2InferenceEngine` (`raas2_engine.py`), `RaaSPool`
    (`raas_pool.py`).
  - `astraflow/dataflow/dataset/` — Rollout / eval dataset loaders
    (deepscaler, alfworld, webshop, livecodebench, gsm8k, ...).
  - `astraflow/dataflow/tests/` — Dataflow unit tests.
  - `astraflow/train_worker/` — Training engine (swappable). Owns
    PPO actor/critic, FSDP2/Megatron backends, launchers, model
    adapters, recovery/saving.
    - `api/cli_args.py` — Dataclass configs validated by
      Hydra/OmegaConf. Source of truth for CLI options and YAML
      schemas. `api/io_struct.py` holds runtime structs such as
      `GenerationHyperparameters`.
    - `engine/` — Training/inference engines (FSDP2, Megatron,
      PPO actor/critic, SFT).
    - `launcher/` — `local.py`, `ray.py`, `slurm.py` launchers plus
      `rpc/` and vLLM/SGLang server launchers.
    - `models/` — Megatron-Core (`mcore/`) and HF Transformers
      (`transformers/`) adapters.
    - `trainer/` — `AstraFlowPPOTrainer` (`ppo_trainer.py`) and
      related orchestration.
    - `platforms/` — CPU / CUDA / NPU abstractions.
    - `tools/` — Developer utilities (e.g. validation, profiling).
    - `utils/` — Cross-cutting helpers: logging, stats, data,
      saver, recover, megatron helpers, FSDP helpers.
  - `astraflow/raas/` — RaaS (Remote Agentic Serving). Launches
    vLLM/SGLang servers, exposes HTTP endpoints for rollout
    generation and weight updates.
    - `server/` — Manager, TCP receiver, FastAPI app.
    - `engine/` — Remote inference engine adapters.
    - `api/cli_args.py` — RaaS-side dataclass configs.
  - `astraflow/workflow/` — Rollout workflows and reward functions
    (swappable).
    - `api/` — Base interfaces: `RolloutWorkflow`
      (`workflow_api.py`), `AsyncRewardWrapper` (`reward_api.py`).
    - `impl/` — Concrete workflows: `rlvr`, `multi_turn`,
      `vision_rlvr`, `solve_and_verify`, `actor_and_verify`,
      `code_*`, `plan_and_solve`, `sm_lg_router`, etc.
    - `impl/agentbench/`, `impl/asearcher/` — environment-specific
      workflow families.
    - `reward/` — Reward callables (`math_verify`,
      `livecodebench_reward`, `human_eval_reward`, `geometry3k`,
      `clevr_count_70k`).
    - `registry.py` — Decorator-based registries for workflows
      and rewards (`WORKFLOW_REGISTRY`, `REWARD_REGISTRY`).
    - `__init__.py` — Imports every `impl/` and `reward/` module so
      the registration decorators run at import time.
  - `astraflow/weight_manager/` — Weight transport (TCP/ZMQ)
    between trainer and RaaS. `transfer/tests/` holds its unit tests.
  - `astraflow/config/` — Hydra/OmegaConf config loader and merging.
- `astraEnv/` — Vendored environment code (not part of the package):
  `AgentBench` (alfworld, webshop), `ASearcher` (retrieval-augmented
  search), `human-eval`. Carries upstream licenses; treat as a
  read-only dependency unless coordinating with maintainers.
- `examples/` — Runnable training recipes grouped by task type
  (`math/`, `code/`, `math-multi-agent/`, `code-multi-agent/`,
  `alfworld/`, `webshop/`, `search/`, `math-efficient-data/`), plus
  shared helpers in `_common/` and `launch_trainer.py`. Each recipe
  ships a `yaml/` directory of configs and a `scripts/` directory of
  numbered launch scripts.
- `docs/` — Sphinx sources. English pages under `docs/en/`
  (architecture / recipes / get-started / developer-guide /
  references). CLI reference is generated by
  `docs/generate_cli_docs.py`.
- `docker/` — `Dockerfile.sglang`, the published image (astraflow +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Infini-AI-Lab/astraflow](https://github.com/Infini-AI-Lab/astraflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

---
trigger: always_on
description: <!-- Think of this file as the go-to brief for AI coding agents working on AReaL. -->
---

<!-- Think of this file as the go-to brief for AI coding agents working on AReaL. -->

# AGENTS.md — AReaL Agent Operations Guide

## TL;DR for coding agents

- **Runtime**: Designed for distributed GPU clusters (FSDP/Megatron + NCCL). Assume
  containerized execution; no standalone local runs.
- **Environment**: Platform images and launcher specs live under `launcher/` and
  docs—reference them instead of hand-rolling virtualenvs.
- **Testing**: Integration and performance tests require multi-node hardware. Explain
  skips explicitly when you cannot access the cluster.
- **Formatting**: Project uses Black/isort/autoflake (see `pyproject.toml`). Surface any
  formatting gaps if you cannot run the tools yourself.
- **Docs**: Source lives under `docs/` (Jupyter Book). Coordinate doc edits with the
  docs build pipeline.

When unsure, leave a `TODO(agent)` comment and note the constraint in your response.

## Repository map

| Path                     | Purpose                                                                        |
| ------------------------ | ------------------------------------------------------------------------------ |
| `areal/api/`             | Core contracts: workflows, engines, CLI configs, IO structs, scheduler APIs.   |
| `areal/workflow/`        | Rollout/agent implementations (`multi_turn`, `rlvr`, `vision_rlvr`).           |
| `areal/engine/`          | Training backends (FSDP2, Megatron, PPO actors) and inference adapters.        |
| `areal/dataset/`         | Dataset loaders & utilities that feed rollouts.                                |
| `areal/reward/`          | Built-in reward functions plus helpers (math parsing, CLEVR counting).         |
| `areal/utils/`           | Logging (`stats_tracker`), tensor helpers, recovery, evaluation, device utils. |
| `examples/`              | Runnable entrypoints for math, multi-turn, RLHF, VLM, search agents.           |
| `areal/launcher/`        | Entry scripts for local, Ray, and Slurm orchestration.                         |
| `docs/`                  | Published docs (https://inclusionai.github.io/AReaL/).                         |
| `realhf/`                | Legacy integrations retained for reference (read-only).                        |
| `functioncall/`          | Tool-calling utilities reused in workflows.                                    |
| `areal/platforms/`       | Cluster abstractions used by advanced agents.                                  |
| `tests/`                 | Pytest suites (many require GPUs or mocked engines).                           |
| `Dockerfile`, `Makefile` | Container recipe and helper tasks (`make docs`, `make lint`).                  |

### Where to find things

- **`areal/api/`** – Contracts for engines, schedulers, dataloaders, and CLI configs.
  Start here when adding new dataclasses or API surfaces.
- **`areal/workflow/`** – Concrete rollout agents (`multi_turn`, `rlvr`, `vision_rlvr`).
  Each illustrates how `RolloutWorkflow.arun_episode` should orchestrate inference and
  rewards.
- **`areal/engine/`** – Training and inference engines: FSDP2, Megatron, PPO actors, and
  SGLang/vLLM adapters. Keep weight versioning logic consistent across edits.
- **`areal/dataset/`** – Stateful data pipeline utilities. New datasets should plug into
  these loaders for replay-safe iteration.
- **`areal/reward/`** – Reward functions and math parsers. Wrap slow logic with
  `AsyncRewardWrapper` in `areal/api/reward_api.py`.
- **`areal/utils/`** – Cross-cutting helpers (logging, stats, tensor containers,
  recovery, evaluation). Prefer reusing these utilities over duplicating logic.
- **`examples/`** – End-to-end wiring scripts for math, multi-turn, RLHF, VLM, and
  search agents. Use them as references for config wiring and launcher usage.
- **`docs/`** – Jupyter Book source; mirrors the high-level architecture and
  customization guides published at https://inclusionai.github.io/AReaL/.
- **`areal/launcher/`** – Orchestration entrypoints (local, Ray, Slurm) plus container
  specs; essential for understanding deployment expectations.
- **`realhf/`** – Legacy integrations retained for reference. Treat this directory as
  read-only unless explicitly extending backward compatibility.
- **`functioncall/` & `areal/platforms/`** – Tool-calling scaffolding and cluster
  abstractions used by advanced agents.

## Distributed operations & tooling

- **Clusters & containers**: Launch configurations live under `areal/launcher/` (local,
  Ray, Slurm). Each entrypoint documents the scheduler expectations; reuse those specs
  instead of inventing ad-hoc run scripts.
- **Shared images**: Platform-specific container images and startup scripts are defined
  alongside launcher configs. Reference them or note when they are missing—do not
  attempt to rebuild CUDA/driver stacks inline.
- **Secrets & endpoints**: Credentials for remote inference (SGLang, vLLM, Redis, etc.)
  are managed outside the repo. Flag their absence rather than hardcoding replacements.
- **Testing limitations**: End-to-end tests (FSDP, Megatron, distributed RPC) require
  multi-node NCCL clusters. If you cannot execute them, state that your validation is
  limited to static analysis/doc updates.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GreatX3/ProAct](https://github.com/GreatX3/ProAct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

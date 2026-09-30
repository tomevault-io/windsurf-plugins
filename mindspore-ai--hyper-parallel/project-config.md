---
trigger: always_on
description: HyperParallel is an **easy-to-use, high-performance distributed parallel acceleration library** for distributed model training, inference and reinforcement learning. It provides unified abstractions for DP, FSDP/HSDP, TP, EP, CP, PP, activation checkpoint/swap, and parameter/optimizer offload. Hybrid strategies combine freely.
---

# HyperParallel — AGENTS.md

## Project Overview

HyperParallel is an **easy-to-use, high-performance distributed parallel acceleration library** for distributed model training, inference and reinforcement learning. It provides unified abstractions for DP, FSDP/HSDP, TP, EP, CP, PP, activation checkpoint/swap, and parameter/optimizer offload. Hybrid strategies combine freely.

Primary target hardware: **Ascend NPU and Nvidia GPU**. Primary framework: **PyTorch**.

---

## HyperParallel-RL Entry

- For RL-owned code, tests, docs, or agent rules, start with [`.agent/rules/hyper-rl.md`](.agent/rules/hyper-rl.md). It is the sole RL entry.
- RL rules do not apply to other HyperParallel modules. Handle a required main-project change separately under that module's rules.
- Use the [RL architecture](docs/rl-architecture.md), [feature navigation](docs/rl-navigation.md), and [module map](.agent/rules/rl/module-map.md) to locate boundaries, implementation/test traces, and ownership.
- RL unit tests live in `tests/ut/rl/`, system tests in `hyper_parallel/rl/tests/st/`, and shared recipes in `tests/common/rl_st_cases.py`. RL ST is run explicitly and is temporarily outside the main-project PR gate. Follow the repository testing rules as well.

---

## Dev Commands

```bash
# Editable install (required so edits to this repo are what Python imports)
cd /path/to/hyper-parallel && pip install -e .

# Verify import path points at this checkout (not a stale worktree / site-packages copy)
python -c "import hyper_parallel, os; print(hyper_parallel.__file__)"

# Unit tests (no multi-card)
pytest -vs tests/ut

# Lint / commit / PR (GitCode fork workflow)
python3 .agent/skills/autogit/scripts/autogit.py check
python3 .agent/skills/autogit/scripts/autogit.py commit -m "feat: ..."
python3 .agent/skills/autogit/scripts/autogit.py pr

# AGENTS.md Skills/Agents table vs disk (also in: autogit check / autogit commit).
# Non-zero exit is blocking. Run it whenever the diff touches *.md.
# Check changed documentation links separately; this script only checks catalogs.
python3 .agent/scripts/check_agents_catalog.py
```

Distributed ST helpers: `torchrun_case()` via `tests.common.distributed_launcher`, or `parallel_case` (see `.agent/rules/testing.md`).

---

## Env Gotchas / Do Not

- **Editable must track this tree.** If `pip show hyper_parallel` shows another path (e.g. a deleted `.worktrees/...`), re-run `pip install -e .` from repo root. Do not rely on `PYTHONPATH` alone.
- **Torch-only.** The `platform/` abstraction layer (and with it the MindSpore backend) has been
  removed. Every module uses native Torch APIs (`torch.distributed`, `torch.Tensor`, autograd)
  directly; do not reintroduce Platform dispatch or MindSpore implementations anywhere.
- **Collectives.** Use `torch.distributed` directly, or the thin wrappers in
  `hyper_parallel/core/context_parallel/utils.py` and `hyper_parallel/core/dtensor/_utils.py`.
- **Never** invent Jenkins build numbers or force-push shared branches in agent workflows.
- Hard distributed rules (canonical): `.agent/rules/project-overview.md` + `.agent/rules/distributed.md` — do not restate long-form elsewhere; link instead.

---

## Key Modules

| Module | Location | Purpose |
|--------|----------|---------|
| **RL** | `hyper_parallel/rl/` | Synchronous LLM RL runtime with Qwen3, GRPO and PPO |
| **DTensor** | `core/dtensor/` | Local shard + DeviceMesh + Placements; redistribution cache |
| **Shard** | `core/shard/` | `custom_shard` / YAML ops + `parallel_*.py` |
| **Tensor parallel** | `core/tensor_parallel/` | `parallelize_module()`, `ParallelStyle`, mesh context |
| **FSDP / HSDP** | `core/fully_shard/` | Param shard/unshard; HSDP under same tree (`hsdp_*.py`) |
| **Pipeline** | `core/pipeline_parallel/` | Torch-only stage schedule, micro-batch, P2P |
| **Activation** | `core/activation_checkpoint/`, `core/activation_memory/` | SAC + activation swap |
| **Checkpoint** | `core/distributed_checkpoint/` | Distributed save/load |
| **Multicore** | `core/multicore/` | Torch-only component with private SHMEM and native build |
| **Collectives** | `collectives/cc.py` | Process groups |
| **Tests** | `tests/ut/`, `tests/torch/` | UT + distributed ST |

---

## Coding Conventions

> Full details: `.agent/rules/code-style.md` (global hard constraint).

- Apache 2.0 header on `.py` (lines 1–16); PEP 8 / ~120 cols; Google-style docstrings; type hints on public APIs
- Imports at module top; the platform-backend lazy-import exception no longer applies
- Load `code-style.md` before generate / edit / commit / review; auto-fix before proceeding

---

## Testing

> Full details: `.agent/rules/testing.md`. How-to for new UT: skill `add-unit-test`.

- **Runner:** pytest + `@arg_mark` (`tests/common/mark_utils.py`)
- **Authoring:** prefer `unittest.TestCase` where existing UT does (pytest still runs them)
- UT: `tests/ut/` — no distributed setup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mindspore-ai/hyper-parallel](https://github.com/mindspore-ai/hyper-parallel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

---
trigger: always_on
description: Instructions for coding agents working in `ambench`, an Isaac Lab extension for aerial manipulation.
---

# AGENTS.md

Instructions for coding agents working in `ambench`, an Isaac Lab extension for aerial manipulation.

## Start Here

Read local context before making structural changes:

1. `README.md`
2. `CODING_STYLE.md`
3. The nearest matching implementation in the same package
4. The public contributor guides:
   - [Extend AM-Bench](https://ambench.github.io/docs/extend/)
   - [Tasks](https://ambench.github.io/docs/extend/task/)
   - [Controllers](https://ambench.github.io/docs/extend/controller/)
   - [Robots](https://ambench.github.io/docs/extend/robot/)
   - [Policies](https://ambench.github.io/docs/extend/policy/)
   - [Workflows](https://ambench.github.io/docs/workflows/)

Prefer repository-consistent changes over clever rewrites.
Follow `CODING_STYLE.md` for code style.

## Quick Commands

Use the lightest command that proves the change works.

Efficient context gathering:

- Prefer `rg` or `rg --files` for codebase search.
- When reading several unrelated files, run independent reads in parallel where the agent runtime supports it.
- Avoid noisy chained shell output for context gathering; separate file reads are easier to review and summarize.

GPU capacity check:

- Before starting substantive work that may use Isaac Sim, rendering, or GPU-heavy validation, run `nvidia-smi` to inspect available GPUs, current memory pressure, and active processes on the machine you are using.
- If multiple GPUs are available, spread heavy runs across them when practical instead of stacking all jobs onto one device, leverage `CUDA_VISIBLE_DEVICES` to isolate workloads, and monitor VRAM usage to avoid overcommitment.
- If only one GPU is available, do not launch multiple heavy Isaac Sim or rendering workloads concurrently. Prefer one active heavy run at a time, because overlapping jobs can exhaust VRAM and destabilize the workstation.
- If desktop applications are already consuming significant VRAM, bias toward lighter validation settings such as fewer environments, headless mode, no extra camera sensors, or a single targeted repro script.

Python environment:

- Prefer the Isaac Lab virtual environment at `../IsaacLab/env_isaaclab`.
- Before running Isaac-dependent commands from `ambench/`, activate it once with `source ../IsaacLab/env_isaaclab/bin/activate`.
- After activation, use plain `python ...` for repo scripts.
- Running `../IsaacLab/env_isaaclab/bin/python` directly without activation may miss the Isaac Sim environment setup that activation provides.
- Do not use `../IsaacLab/_isaac_sim/python.sh` or other kit-Python entrypoints for normal repo validation from `ambench/`; they can miss the packages installed in `env_isaaclab` and produce misleading import/startup failures.
- For Isaac Lab or Isaac Sim Python scripts that touch `isaaclab`, `pxr`, or Omniverse app state, prefer files that use the standard `AppLauncher` bootstrap before the rest of the script runs.
- For ad hoc Isaac-related snippets, activate `env_isaaclab`, run plain `python`, and initialize `AppLauncher` before importing or using `pxr`/Isaac Sim APIs.

Install package(s):

```bash
uv pip install -e source/ambench
uv pip install -e source/ambench_learn
```

List registered environments:

First activate the environment once:

```bash
source ../IsaacLab/env_isaaclab/bin/activate
```

Then run commands with `python`:

```bash
python scripts/environments/list_envs.py
```

Smoke-test an environment:

```bash
python scripts/environments/zero_agent.py --task <env-id>
```

`zero_agent.py` is a continuous runner. For bounded validation, wrap it in `timeout`, confirm that scene setup/reset/stepping started, and then ensure no child Isaac process remains before reporting completion.

Interactive inspection:

```bash
python scripts/environments/teleop_se3_agent.py --task <env-id>
```

Validate recorded demos:

```bash
python scripts/data/validate_lerobotdataset.py ...
```

Run formatting/hooks:

```bash
pre-commit run --all-files
```

If Isaac Sim, Isaac Lab, GUI, GPU, or datasets are unavailable, say exactly what you could not verify.

## Release Boundary

Release pruning removed the non-benchmark tasks and their assets, the internal
experiment and demo scripts, and the internal planning and tooling directories.
The task registry is now exactly the 12 task families described in the paper.
Do not reintroduce that material without a deliberate decision.

The registry is the authority on what ships: `python scripts/environments/list_envs.py`
prints it, and the [Environment Registry](https://ambench.github.io/docs/reference/environments/)
is the written reference.

Generated artifacts (datasets, checkpoints, videos, wandb runs, acados build
output) are covered by `.gitignore` and must stay untracked.

Areas that require deliberate review before changing their public support boundary:

- `source/ambench/ambench/utils/camera_utils.py`
  Reason: camera support and its public support boundary need file-level review.
- `source/ambench/ambench/utils/image_processing.py`
  Reason: image-processing support and dependencies need file-level review.
- `ext/`
  Reason: optional dependencies are curated individually; do not treat the directory as a monolith.

## Tech Stack And Constraints

- Python `>=3.11`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ambench/ambench](https://github.com/ambench/ambench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

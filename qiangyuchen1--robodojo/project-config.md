---
trigger: always_on
description: RoboDojo is a unified sim-and-real robotics benchmark built on NVIDIA Isaac Sim / IsaacLab. Policy evaluation runs via XPolicyLab and the shared simulation stack.
---

# RoboDojo — Claude Code Guide

## Project Overview

RoboDojo is a unified sim-and-real robotics benchmark built on NVIDIA Isaac Sim / IsaacLab. Policy evaluation runs via XPolicyLab and the shared simulation stack.

Contributors maintain **Tasks** (scene + reward logic), shared **env** / **env_cfg** configs, and eval infrastructure under **scripts/** and **src/eval_client/**.

Human-facing docs: https://robodojo-benchmark.com/doc/

## Repository Layout

```
task/RoboDojo/
  config/<task_name>.yml     task scene/object config
  tasks/<task_name>.py       task logic (inherits TaskEnv)
  task_registry.py           dynamic task loader
  demos/                     non-inventory demos (e.g. dlc)
env/                         TaskEnv backbone and managers
env_cfg/                     robot / scene / sim / camera YAML
src/eval_client/             eval client (main.py, eval_env.py)
utils/                       paths, save/load, pipeline helpers
scripts/                     robodojo.sh, install.sh, init_assets.sh, internal/
docker/                      container eval (see docker/README.md)
XPolicyLab/                  policy server + deploy (submodule)
third_party/                 IsaacLab, curobo (submodules)
Assets/                      robot/object assets (scripts/init_assets.sh; not in git)
eval_result/                 runtime eval output
```

## Submodules & Ownership

| Own in RoboDojo | Own in XPolicyLab |
| :-- | :-- |
| `env/`, `env_cfg/`, `task/RoboDojo/`, `src/eval_client/` | Policy code, checkpoints, `deploy.yml`, policy server |
| `scripts/robodojo.sh`, `scripts/eval_policy.sh` | `XPolicyLab/policy/<POLICY>/eval.sh`, `setup_eval_*` scripts |
| `utils/`, install/assets scripts | Policy-specific dependencies and training |

- Submodule URL: `https://github.com/XPolicyLab/XPolicyLab.git` (branch `main`).
- Update the gitlink pin intentionally; do not edit submodule contents unless updating the pin.
- RoboDojo eval client uses `XPolicyLab/client_server/ws/model_client.py` (WebSocket transport).

## Eval Commands & Flow

**CLI** (`bash scripts/robodojo.sh <command>`):

| Command | Purpose |
| :-- | :-- |
| `doctor` | Check assets, configs, env before eval |
| `eval` | One task: policy server + sim client on localhost |
| `server` | Policy server only (split / multi-machine eval) |
| `client` | Sim client only (connect to running server) |
| `smoke` | Selected/all tasks, `EVAL_NUM=1` by default |
| `benchmark` | Full sequential sweep (`--eval-num native` or numeric) |
| `summarize` | Aggregate `eval_result/` into markdown table |
| `tasks` | List runnable tasks (maintainer inventory) |

**Single-machine eval:**

```
robodojo.sh eval
  → scripts/internal/run_policy_eval.sh
    → (cd policy_dir) setup_eval_policy_server.sh   # policy server
    → setup_eval_env_client.sh                    # sim client → src/eval_client/main.py
```

- Policy `eval.sh` must exist under `--policy-dir` (used for validation and optional `expert_num`); `robodojo eval` does **not** invoke `eval.sh` directly.
- Policy `setup_eval_*` scripts run with **CWD = policy directory** (relative paths in policies).
- Default transport: **WebSocket** (`protocol: ws` in policy `deploy.yml`; see `scripts/eval_policy.sh`).

**Split eval:**

```
robodojo.sh server  →  scripts/internal/run_policy_server.sh
robodojo.sh client  →  scripts/eval_policy.sh  →  src/eval_client/main.py
```

**Docker:** sim client in container, policy server on host — see `docker/README.md`.

Fast validation:

```bash
bash scripts/robodojo.sh doctor --skip-isaac --skip-conda --skip-policy
bash scripts/robodojo.sh eval --policy-dir XPolicyLab/policy/<POLICY> --task stack_bowls --ckpt <CKPT> --policy-env <ENV> --dry-run
bash scripts/robodojo.sh smoke --policy-dir XPolicyLab/policy/<POLICY> --ckpt <CKPT> --policy-env <ENV> --only stack_bowls,push_T --dry-run
```

Full eval-infrastructure acceptance is **sequential**, not parallel:

```bash
bash scripts/robodojo.sh smoke --policy-dir XPolicyLab/policy/<POLICY> --ckpt <CKPT> --policy-env <ENV> --fail-fast
```

New code imports from `env.*`.

## Tasks

- **54 runnable task configs** (42 base + 12 `_random` generalization variants).
- Registration is dynamic: `task_registry.load_task_class(task_name)` imports `task.RoboDojo.tasks.<task_name>` and expects class name == module name.
- Config path: `task/RoboDojo/config/<task_name>.yml`.
- Inventory: `bash scripts/robodojo.sh tasks` or `python scripts/internal/task_inventory.py --format json --check`.
- `task/RoboDojo/demos/` (e.g. `dlc`) is outside the benchmark inventory scan.

## Naming

- **Config / scripts**: lowercase `snake_case` — `env_cfg/`, `utils/`, `scripts/`
- **Python packages/directories**: `snake_case` — `env/robot_manager/`, `env/scene_manager/objects/`
- **Python module files**: `snake_case.py` — `task_env.py`, `obs_manager.py`
- **Python classes**: **PascalCase** — `RewardManager`, `TaskEnv`
- **Tasks**: filename, YAML name, exported env class, and layout/result paths must match. Most tasks use lowercase `snake_case`; **`play_Xylophone`**, **`swap_T`**, **`push_T`**, and **`push_T_random`** match uppercase asset names.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QiangyuChen1/RoboDojo](https://github.com/QiangyuChen1/RoboDojo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

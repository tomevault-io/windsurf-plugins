---
trigger: always_on
description: This repository contains Wojtek's ROS stack (`ros/`) and its MuJoCo MJX/Brax
---

# Wojtek agent guide

This repository contains Wojtek's ROS stack (`ros/`) and its MuJoCo MJX/Brax
training project (`training/`). On supported CUDA GPUs, MJX uses the **MJWarp**
backend; its JAX backend is the CPU/non-MJWarp fallback. Brax provides PPO and
training wrappers, not the physics engine. Treat training configuration as an
explicit, reviewable experiment input: resolve it before launching a run and
keep the command, seed, and resulting run directory together.

## Repository map and boundaries

- `training/` is the Hydra-configured MuJoCo MJX/Brax PPO project, with
  **MJWarp** (the Warp implementation of MJX) as its primary CUDA backend.
  `task.env.sim.backend=auto` selects `warp` on a supported GPU and `jax`
  otherwise. Its generated outputs belong in ignored `training/runs/`,
  `training/videos/`, caches, and WandB directories—not in commits.
- `ros/` is the deployment and robot-control workspace.  Treat real-robot
  launch and policy deployment as human-authorized, safety-sensitive actions.
- `ros/src/wojtek_description/mujoco/wojtek.xml` is the model source;
  `wojtek_mjx.xml` and `scene_mjx.xml` are generated via `./training/run.sh
  build`.  Do not hand-edit generated XML.
- `skills/` contains opt-in local guides.  Claude users must explicitly
  symlink a skill or ask to read its `SKILL.md`; see [skills/README.md](skills/README.md).
- Use **Wojtek** everywhere: prose, artifact names, run names, and new paths.
  `fbb-*` survives only where it is a literal identifier of an existing
  configuration or published artifact.
- Remote execution and machine provisioning live in a separate private
  operations repository; this repository holds only cluster-agnostic job
  payloads under `training/jobs/`.
- This repository is public.  Never commit credentials or the identity of
  private infrastructure: passwords, tokens, keys; hostnames, IPs, logins,
  or port maps of private machines; cluster site, account, grant, or
  partition names; personal emails or aliases.  Such values enter at run
  time through an environment variable declared at the top of the script
  and filled from the gitignored `.env` (see `.env.example`) — the
  `UBUNTU_PRO_TOKEN` and `WOJTEK_AP_PSK` handling in `ros/deploy.sh` is
  the pattern to copy.  A script that targets one concrete private machine
  does not belong here at all; it goes to the operations repository.
  `training/tests/unit/test_job_scripts.py` enforces this for job
  payloads; the rule covers the whole tree, comments and docs included.
- New code is Apache-2.0 by default; do not pick another license unless the
  author explicitly asks for one.  Declare it as `<license>Apache License
  2.0</license>` in `package.xml` and `license="Apache-2.0"` in
  `setup.py`/`pyproject.toml`.  The only exception is the GPL-licensed IMU
  firmware directories; see `NOTICE`.

## Start with the training reference

- [Training configuration reference](training/docs/configuration.md) — every
  Hydra group, root setting, task setting, domain-randomization switch, PPO
  setting, experiment preset, and command-mode usage.  Its "Course benchmark"
  section defines the path-following scores and the frozen follower constants
  that must not be retuned.
- [Training lessons](skills/brax-locomotion-training/references/wojtek-training-lessons.md)
  — evidence from previous locomotion iterations; consult it before changing
  rewards, observations, or gait behavior.
- [MJWarp backend report](docs/plans/mjwarp-phase0-report.md) — backend buffer
  sizing and validation context.
- [Demo guide](training/demo/README.md) — interactive navigation demo usage.
- [SCAN-Planner](training/docs/scan-planner.md) — the collision-aware local
  planner that executes the VLM's mid-level commands; read it before touching
  `wojtek_rl/scan/`, the room demo's executor, or the nav-eval collision
  metrics.

## Safe training workflow

Run commands from the repository root.  `training/run.sh` changes into the
training project before invoking Python.

```bash
# Inspect the resolved Hydra configuration/overrides; this does not train.
./training/run.sh train --cfg job --resolve

# Resolve an intended change before spending GPU time.
./training/run.sh train +experiment=locomotion \
  ++ppo.num_timesteps=300000000 --cfg job --resolve

# The train/checkpoint path is checked with a short bounded run on a GPU
# box before the full budget.  CPU smoke runs are not viable (MJX compile
# on CPU); do not run them.  A preset can override a short run's PPO
# budget, so pin ++ppo.num_timesteps when selecting one.
```

Rules for agents changing or launching training:

- Use the detailed reference rather than guessing Hydra paths.  Code-defaulted
  environment fields require `+` or `++` on the command line; `++` is the
  safest form when a key may or may not already be present in YAML.
- Run `--cfg job --resolve` first, then a GPU model check
  (`./training/run.sh check --gpu --backend warp`) and a bounded train before
  a full budget. Do not claim a reward change is successful without the fixed
  evaluation battery or report.
- Actor observations must be available on the physical robot.  Observation
  layout changes invalidate checkpoints and deployment artifacts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [machinekind/w01-tek](https://github.com/machinekind/w01-tek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->

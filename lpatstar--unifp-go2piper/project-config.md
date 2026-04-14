---
trigger: always_on
description: This repository is a Go2+Piper-oriented fork of UniFP for whole-body position-force control in Isaac Gym.
---

# AGENTS.md

## Project Identity

This repository is a Go2+Piper-oriented fork of UniFP for whole-body position-force control in Isaac Gym.

Primary task:
- `go2_piper_pos_force`

The repo is not a clean greenfield project. It keeps a large amount of upstream UniFP / B2+Z1 structure, and the Go2+Piper task is layered on top of that shared implementation.

## Environment Assumptions

- Python `3.8`
- Isaac Gym Preview 4
- Linux, typically Ubuntu `20.04` or `22.04`
- PyTorch and CUDA configured separately before running scripts

This repo is not meaningfully testable in a generic sandbox without a working Isaac Gym install and GPU runtime.

## Main Entry Points

Train:
```bash
cd legged_gym/scripts
python train_go2piperposforce.py --task=go2_piper_pos_force --headless
```

Play:
```bash
cd legged_gym/scripts
python play_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name>
```

Play with joint tracking draw export:
```bash
cd legged_gym/scripts
python play_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name> --draw
```

Keyboard teleop / manual inspection:
```bash
cd legged_gym/scripts
python keyplay_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name>
```

Automated evaluation:
```bash
cd legged_gym/scripts
python eval_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name> --headless
```

## Code Map

Go2-specific config:
- `legged_gym/envs/go2/go2_piper_pos_force_config.py`

Go2 task alias:
- `legged_gym/envs/go2/legged_robot_go2_piper_pos_force.py`

Shared environment implementation that actually contains most runtime logic:
- `legged_gym/envs/b2/legged_robot_b2z1_pos_force.py`

Shared base config inherited by Go2:
- `legged_gym/envs/b2/b2z1_pos_force_config.py`

Training stack:
- `legged_gym/b2_gym_learn/ppo_cse_pf/`

Task registration:
- `legged_gym/envs/__init__.py`
- `legged_gym/utils/task_registry.py`

## Architectural Notes

- `legged_robot_go2_piper_pos_force.py` is only a thin alias to the shared B2+Z1 environment class.
- Real Go2+Piper behavior is split between:
  - Go2 config overrides in `go2_piper_pos_force_config.py`
  - shared control / observation / viewer / force logic in `legged_robot_b2z1_pos_force.py`
- The Go2 config keeps the upstream control layout:
  - 12 leg joints plus the first 5 arm joints are policy-controlled
  - the final 3 Piper wrist / gripper joints are fixed-PD controlled
- Many changes that look "Go2-specific" still belong in the shared B2 environment because that is where command handling, force injection, debug drawing, viewer hotkeys, and rollout behavior live.

## Force And Command Conventions

- Commanded force and externally applied force are separate paths.
- Commanded EE force and commanded base force are stored in `commands`.
- External disturbance forces are applied through `self.forces`.
- In `play`, green arrows represent commanded force and blue arrows represent applied external force.
- For Go2+Piper, gripper force ranges are narrower than the inherited B2+Z1 defaults.

## Script Behavior Notes

- `train_go2piperposforce.py` delegates to the shared B2/Z1 training implementation and forces `args.task = "go2_piper_pos_force"` when not provided.
- `play_go2piperposforce.py` delegates to the shared play implementation and enables force visualization / play-side command-force behavior through module flags.
- `play_go2piperposforce.py --draw` runs a short rollout, saves joint command-vs-actual plots for one front-left leg and the task-relevant arm joints, then exits. The shared play script auto-resolves the correct arm joint names for Go2+Piper and B2+Z1.
- `keyplay_go2piperposforce.py` creates a single-env teleop setup, disables most randomization, and relies on viewer keyboard events for command updates.
- `eval_go2piperposforce.py` is the main reproducible checkpoint benchmark. Prefer it over ad hoc `play` sessions when comparing runs.
- `play_go2piperposforce.py`, `keyplay_go2piperposforce.py`, and `eval_go2piperposforce.py` all use checkpoint-resume loading. If `--load_run` and `--checkpoint` are omitted, the shared loader resolves to the latest run directory and then the latest saved `model_*.pt` checkpoint inside it.

## Play Draw Notes

- `play --draw` is the standard joint-level diagnostic path for tuning analysis.
- Default command:
  - `python play_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name> --draw`
- Useful optional flag:
  - `--draw_steps <N>` to control how many play steps are recorded before the script exits
- Output directory:
  - `play_draws/`
- Expected files:
  - one leg joint tracking plot
  - one arm joint tracking plot
- During tuning, inspect the PNG plots directly and judge both:
  - whether actual tracks command
  - and whether command itself already shows obvious oscillation or other bad control patterns
- When a tuning sample does not already have draw plots, generate them before forming a tuning conclusion.

## Keyplay Notes

- `keyplay` intentionally disables random command resampling and random force events.
- Keyplay controls are split between:
  - `legged_gym/scripts/keyplay_go2piperposforce.py`
  - viewer-event handling inside `legged_gym/envs/b2/legged_robot_b2z1_pos_force.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LPatstar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

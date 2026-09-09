---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`i2rt` is a Python client library for I2RT hardware: YAM 6-DOF arms, their grippers, and the Flow Base
mobile platform. It talks to Damiao (DM) series motors over a CAN bus, provides MuJoCo simulation and
gravity compensation, and ships URDF/MJCF robot models. The package is named `i2rt` and built with **Flit** from
`pyproject.toml`; the root `setup.py` is legacy/unused (stale `version`, minimal deps) — treat
`pyproject.toml` as authoritative for dependencies and metadata.

## Environment & commands

The project uses **uv** (not pip/venv directly). Python 3.11 is what CI runs; `requires-python >= 3.10`.

```bash
# One-time setup
uv venv --python 3.11 && source .venv/bin/activate
uv pip install -e .              # or: uv sync --dev  (installs dev tools too)

# Tests (all run in MuJoCo sim — no CAN bus / motors needed)
uv run pytest -n auto            # full suite, parallel (what CI runs)
uv run pytest i2rt/robots/tests/test_urdf_mjcf_alignment.py -v      # one file
uv run pytest -k "yam_pro and linear_4310"                          # by keyword

# Lint / format / type-check (match CI + pre-commit exactly)
ruff check .                     # CI pins ruff==0.15.6; line-length 119, ANN rules on
ruff format .
uv run python -m pyright         # pre-commit only type-checks a scoped allowlist (see below)
pre-commit run --all-files       # uv-lock, uv-sync --locked, ruff, ruff-format, pyright, nbstripout
```

Two GitHub Actions workflows gate PRs: `ruff.yml` (`ruff check`) and `unit_tests.yml` (`uv sync --dev`
then `uv run pytest -n auto`). Both use Python 3.11.

### Tooling conventions

- **All CLIs use `tyro`, never `argparse`.** Every `.py` entry point (in `scripts/`, `i2rt/`, `examples/`)
  parses args with `tyro.cli(...)`. Follow this when adding or editing a CLI.
- **Type-checking is split:** `pyproject.toml` has a `[tool.mypy]` section (targets 3.12, excludes
  `scripts/`), but the pre-commit hook actually runs **pyright**, scoped to a small allowlist
  (`i2rt/utils/(encoder_manager|can_flash).py`). Extend the hook's `files:` regex as more modules are cleaned up.
- Ruff's `ANN` (annotations) rules apply to `scripts/` even though mypy excludes that directory.

## Architecture

### The Robot abstraction (`i2rt/robots/`)

Everything downstream programs against the `Robot` **Protocol** in [robot.py](i2rt/robots/robot.py)
(`get_joint_pos`, `command_joint_pos`, `get_observations`, ...). There are two implementations:

- **`MotorChainRobot`** ([motor_chain_robot.py](i2rt/robots/motor_chain_robot.py)) — real hardware. Runs a
  background control thread over the CAN motor chain, does gravity compensation using a MuJoCo model of the
  robot, and handles gripper force limiting / calibration.
- **`SimRobot`** ([sim_robot.py](i2rt/robots/sim_robot.py)) — MuJoCo-only, in-memory joint state. Implements the
  same protocol so tests and visualizers use it interchangeably. **This is the path the entire test suite exercises.**

Both are constructed through the single factory `get_yam_robot(...)` in
[get_robot.py](i2rt/robots/get_robot.py), switched by a `sim: bool` argument. That factory is where arm +
gripper config, joint limits, gains, and motor offsets get assembled — read it first to understand how a
robot instance comes together.

### Config-driven, runtime-composed models

A robot = an **arm** + a **gripper**, each independently selected. A **hardware revision is its own
arm variant** — `ArmType.YAM_ULTRA_2` (`yam_ultra_2`) is revision 2 of `yam_ultra` — so `ArmType` alone
identifies an arm across the robot-construction API (`get_yam_robot`, `combine_arm_and_gripper_xml`,
`ArmType.get_xml_path`, `_load_arm_config`) and the four robot CLIs (`motor_chain_robot.py` plus the three
`examples/` entry points): none of those take a `version` argument or a `--version` flag. A revision is
still passed explicitly *below* that line, where it is load-bearing and must not be "cleaned up" — the path
resolver `get_arm_xml_path(arm, version)` and the offline pipeline script `sync_gripper_mounts.py --version N`
(both described below). `ArmType.family` / `ArmType.version` unpack a variant into the on-disk
`(family, v<N>)` pair that the config and model loaders resolve paths from. Registering a revision is a
**two-place edit** in [utils.py](i2rt/robots/utils.py): the `ArmType` member **and** its `(family, revision)`
entry in the `_ARM_VARIANTS` map. The two enums are the entry points:

- `ArmType` / `GripperType` enums live in [utils.py](i2rt/robots/utils.py) and map to:
  - a **hardware YAML** in [i2rt/robots/config/](i2rt/robots/config/) — motor IDs, directions, kp/kd,
    gravity-comp factors, gripper limits, force-torque maps. **Arm** configs are version-suffixed
    (`yam_v1.yml`, `yam_ultra_v1.yml`); **gripper** configs are not (`linear_4310.yml`).
  - a **robot model** (URDF + MJCF) under [i2rt/robot_models/](i2rt/robot_models/), resolved via the path
    constants in [i2rt/robot_models/__init__.py](i2rt/robot_models/__init__.py).
- **`combine_arm_and_gripper_xml`** (in [utils.py](i2rt/robots/utils.py)) merges the arm MJCF and gripper MJCF

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [i2rt-robotics/i2rt](https://github.com/i2rt-robotics/i2rt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

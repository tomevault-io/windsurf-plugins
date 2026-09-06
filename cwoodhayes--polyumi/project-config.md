---
trigger: always_on
description: validates `package.xml`. Don't re-add the others when generating new ROS packages.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PolyUMI is a multimodal data collection system for robot imitation learning. See [README.md](README.md) for a full description, architecture diagrams, and usage instructions.

## Common Commands

### Linting

Fix everything fixable, in one command:
```bash
uv run ruff check --fix . && uv run ruff format .
```
Fix first, format last, so the formatter always has the last word over any autofix.
To check without writing (what CI runs): `uv run ruff check . && uv run ruff format --check .`

**What that does and does not fix.** Formatting is fully automatic — whitespace, quotes, line
length, trailing commas never need hands. Lint mostly is not: this repo selects `E`, `F`, `D`,
`W`, and the findings are dominated by `D` (pydocstyle), which is unfixable by construction —
ruff cannot invent a docstring. In the b6cfe76 cleanup `ruff format` fixed 20 of 20 files
unaided, while `--fix` resolved 1 of 13 lint errors; the other 12 were missing docstrings and
two hand edits. So expect the command to leave `D1xx` behind for you to write.

Two traps worth knowing:
- `D1xx` only fires on *public* modules — a file named `_foo.py` is private, so nothing in it
  is ever reported missing a docstring.
- `--unsafe-fixes` will clear things like `F841` (unused variable) by deleting the assignment.
  That drops the right-hand side too, so it changes behaviour when the RHS has side effects.
  It is correctly off by default; reach for it per-file, not repo-wide.

ruff is pinned (`[dependency-groups] lint` in pyproject.toml) because its formatter output
changes between minor releases — an unpinned ruff lets two developers reformat the same files
back and forth. Use `uv run ruff`, not `uvx ruff`, so you get the pinned version.
`.github/workflows/lint.yml` enforces this on every PR; it installs only the `lint`
group, since the `dev` group's picamera2 is Pi-only and will not build on a runner.
`external/` is excluded — those are third-party submodules carrying upstream style.

### Tests
```bash
cd pi
pytest test/
# Single test file:
pytest test/files/test_session.py
```

When running ingest-side pytest commands in this workspace, disable pytest plugin autoload to avoid ROS-side import side effects from system site packages:
```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest ingest/test/test_preproc.py
```

**ROS2 tests** need ROS's interpreter (they import `rclpy`), so `uv run` is wrong here — use
`/usr/bin/python3` with the workspace sourced and `VIRTUAL_ENV` unset:
```bash
bash -c 'unset VIRTUAL_ENV; cd ros2_ws && source /opt/ros/kilted/setup.bash \
  && source install/setup.bash && cd src/polyumi_ros2 \
  && /usr/bin/python3 -m pytest test/test_policy_client_node.py -q'
```
`colcon test --packages-select polyumi_ros2` also runs them, and is expected to pass clean:
```bash
bash -c 'unset VIRTUAL_ENV; cd ros2_ws && source /opt/ros/kilted/setup.bash \
  && colcon test --packages-select polyumi_ros2 \
  && colcon test-result --test-result-base build/polyumi_ros2'
```
The **NUC bridges** in `nuc/` are standalone scripts, not an ament package, so `colcon test`
never sees them. `nuc/test_*.py` run on the laptop anyway — they need only the `franka_msgs` /
`moveit_msgs` message definitions (built in `ros2_ws`, or from `/opt/ros`) and mock the service
and action clients, so no hardware, no move_group, and no NUC is involved:
```bash
bash -c 'unset VIRTUAL_ENV; source /opt/ros/kilted/setup.bash \
  && source ros2_ws/install/setup.bash \
  && /usr/bin/python3 -m pytest nuc/ -q'
```

The generated `ament_copyright` / `ament_flake8` / `ament_pep257` linter tests were **deleted** —
their ROS defaults (99 cols, ament import order) contradicted this repo's ruff config, so
`colcon test` failed regardless of whether real tests passed. Python style is ruff's job alone;
`ruff check ros2_ws/` is expected to be clean. Only `ament_xmllint` remains, since nothing else
validates `package.xml`. Don't re-add the others when generating new ROS packages.

### Deploy to Pi
```bash
./deploy.sh <ssh_hostname>   # rsync pi/ + polyumi_pi_msgs to Pi, embeds git hash in _version.py
```

### Pi (run on device)
```bash
polyumi-pi stream
polyumi-pi record-episode --fps 10 --robot polyumi_gripper --task <task_name>
polyumi-pi start-scene --robot polyumi_gripper --task <task_name>
polyumi-pi --help   # full command list
```

### ROS2 (host PC)
```bash
cd ros2_ws
rosdep install --from-paths src --ignore-src -r --rosdistro kilted
colcon build && source install/setup.bash
ros2 launch polyumi_ros2 stream_demo.launch.xml
```

#### FR3 arm (split topology)
The Franka **FR3** is driven from the **NUC** (Ubuntu 22.04, ROS2 Humble, the
Franka stack); the laptop (Ubuntu 24.04, ROS2 Kilted) runs PolyUMI's nodes,
camera, Foxglove, and `policy_client_node`. They interoperate over **CycloneDDS**
(domain 0, `10.0.0.x` link, unicast peers). On the laptop, before launching:
```bash
source setup_franka_env.sh   # RMW=cyclonedds, domain 0, CYCLONEDDS_URI, 10.0.0.1 on enp0s31f6
```
On the NUC, two launch files: `nuc/launch/fr3_bringup.launch.py` (the hardware session —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cwoodhayes/PolyUMI](https://github.com/cwoodhayes/PolyUMI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->

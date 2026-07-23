---
trigger: always_on
description: How to work here. Architecture and inter-package contracts → [ARCHITECTURE.md](ARCHITECTURE.md).
---

# CLAUDE.md — `rosbot_ros` working guide

How to work here. Architecture and inter-package contracts → [ARCHITECTURE.md](ARCHITECTURE.md).

> **Top rule:** draft a short spec + flag sensitive spots **before** coding non-trivial changes.

---

## 1. Context

- ROS 2 **jazzy** (Ubuntu 24.04) packages for Husarion **ROSbot 2/3** and **ROSbot XL** (optional OpenMANIPULATOR-X arm).
- Workspace: `~/Husarion/Workspaces/rosbot_ws/`, repo as `src/rosbot_ros`.
- Main / PR target branch: **`jazzy`** (not `main`).
- Targets: real robot (micro-ROS) **or** Gazebo Harmonic sim — shared URDF + ros2_control.
- Packages: `rosbot` (meta), `rosbot_bringup`, `rosbot_controller`, `rosbot_description`, `rosbot_gazebo`, `rosbot_hardware_interfaces`, `rosbot_joy`, `rosbot_localization`, `rosbot_moveit`, `rosbot_utils`.

---

## 2. Spec before code (mandatory)

For new launches, public args, hardware interfaces, controllers, nodes, URDF/firmware changes — first state:

1. **Goal** (one sentence).
2. **Impact** (packages, public args, firmware/snap compat).
3. **Sensitive spots** — `rosbot_hardware_interfaces` (ABI ↔ firmware), `controllers.yaml` "Based on real measurements" lines, URDF/xacro (MoveIt SRDF regen), public topics in [ROS_API.md](ROS_API.md).
4. **Test plan** (pre-commit, `colcon test`, manual sim/HW run).

15 min alignment beats a re-do after PR review.

---

## 3. Workflow

### 3.1 First-time setup

```bash
cd ~/Husarion/Workspaces/rosbot_ws
vcs import src < src/rosbot_ros/rosbot/rosbot_hardware.repos
vcs import src < src/rosbot_ros/rosbot/rosbot_simulation.repos
export PIP_BREAK_SYSTEM_PACKAGES=1
sudo rosdep init   # only if not previously initialized
rosdep update --rosdistro $ROS_DISTRO
rosdep install -i --from-path src --rosdistro $ROS_DISTRO -y
pip install pre-commit && (cd src/rosbot_ros && pre-commit install)
```

### 3.2 Daily loop (at workspace root)

```bash
source /opt/ros/jazzy/setup.bash
[ -f install/setup.bash ] && source install/setup.bash
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release --packages-select <pkg>
# or --packages-up-to <pkg> after URDF/dep changes
colcon test --packages-select <pkg> && colcon test-result --verbose
source install/setup.bash
```

**Symlink-install** symlinks `launch/`, `config/`, `urdf/`, Python `scripts/` — edit in `src/` works without rebuild. C++ changes still need rebuild.

### 3.3 Pre-commit + commit

`pre-commit run -a` runs all hooks (also fires on `git commit`). Hooks: `black` (LL=99), `isort` (black profile), `flake8` (ignore E501,W503), `clang-format`, `cmake-format`, `codespell`, `doc8` (LL=100), `markdownlint-fix`, `prettier-package-xml`, `sort-package-xml`, `ament_copyright`, `yamlfmt`. **Don't add new formatters without agreement.**

### 3.4 Commits + PRs

- **No direct commits to `jazzy`.** Feature branch + PR.
- PR template has a `## Changelog description` section — fill it.
- Backports via `backport.yaml` after labeling.

### 3.5 Run after build

```bash
ros2 launch rosbot_bringup rosbot.yaml             # ROSbot 2/3
ros2 launch rosbot_bringup rosbot_xl.yaml          # ROSbot XL
ros2 launch rosbot_gazebo simulation.yaml robot_model:=rosbot_xl
# args: README.md "Launch Arguments" or `ros2 launch <pkg> <file> -s`
```

---

## 4. CI / Docker / Devcontainer

- **Devcontainer:** `.devcontainer/compose.yaml` mounts repo at `/home/husarion/ros2_ws/src/rosbot_ros`. `runtime: nvidia` (needs NVIDIA Container Toolkit).
- **Production images:** `docker/Dockerfile.{hardware,simulation}`, built by `build-docker.yaml` on push to `jazzy`. HW drops `rosbot_gazebo`, sim drops `rosbot_bringup`.
- **CI:** `ci.yaml` orchestrates `pre-commit` → `tests.yaml` (`colcon test` over `packages-select-regex: rosbot*`) → (on `jazzy`) `build-docker.yaml`. `rosbot_bringup` tests run with fake HW topics (`hardware_bridge:=False`); `rosbot_gazebo` keeps only offline launch/schema tests (real-Gazebo tests removed).
- **VS Code tasks:** inherited from `panther` repo, `build.sh` uses `--packages-up-to panther` — prefer manual `colcon build` from §3.2.

---

## 5. Conventions

- **License header** (Apache 2.0) in every new source file (`ament_copyright` enforced). Maintainer: `support@husarion.com`.
- **Python:** `black` LL=99, type hints encouraged. Scripts → `<pkg>/scripts/`, installed via `install(PROGRAMS …)` to `lib/${PROJECT_NAME}`. Modules → `ament_python_install_package`.
- **C++:** `clang-format` (default ROS), `-Wall -Wextra -Wpedantic` (+`-Wshadow -Wold-style-cast` in `rosbot_joy`). C++17 min. HW plugins via `pluginlib` in `rosbot_hardware_interfaces.xml`.
- **Launch:** prefer YAML frontend. Use Python only when logic is required (example: [microros.launch.py](rosbot_bringup/launch/microros.launch.py)). Keep arg shape — tests, snaps, sibling packages anchor to `namespace`, `robot_model`, `config_dir`, `mecanum`, `use_sim`.
- **Configs:** every package accepts `config_dir`. Default = `share/<pkg>/config/`; with `config_dir` set, reads `<config_dir>/<pkg>/config/…` (snap relies on this — **don't break it**). Generate external dir: `ros2 run rosbot_utils create_config_dir <dst>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [husarion/rosbot_ros](https://github.com/husarion/rosbot_ros) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

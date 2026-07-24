---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SUAVE (Self-adaptive Underwater Autonomous Vehicle Exemplar) models a single AUV performing a pipeline inspection mission: searching for a pipeline, following it, and inspecting it. The repo cleanly separates a **managed subsystem** (vehicle mission functionality) from **managing subsystems** (adaptation logic), so different adaptation managers can be plugged in through standard ROS 2 interfaces.

Runtime stack: ROS 2 Humble · Gazebo Harmonic · ArduSub/ArduPilot SITL · MAVROS · BehaviorTree.CPP · MROS2/Metacontrol · Docker/Kasm.

## Repository Layout

| Path | Purpose |
|---|---|
| `suave/` | Managed subsystem: functionalities, launch files, sim config |
| `suave_monitor/` | Monitor nodes (thruster, battery, water visibility) — publish diagnostics |
| `suave_missions/` | Mission planners and launch files |
| `suave_metrics/` | Metrics collection |
| `suave_runner/` | Experiment runner and statistical analysis |
| `suave_tools/` | Auxiliary tools (PlotJuggler config, etc.) |
| `suave_msgs/` | Custom ROS service definitions (`Task.srv`, `GetPath.srv`) — `ament_cmake` |
| `suave_managing/suave_none/` | No-manager launch variant |
| `suave_managing/suave_random/` | Random managing subsystem |
| `suave_managing/suave_metacontrol/` | MROS2/Metacontrol managing subsystem |
| `suave_managing/suave_bt/` | BehaviorTree.CPP managing subsystem (C++17) |
| `docker/` | Dockerfile definitions and install scripts |
| `runner/` | Shell scripts for running experiments |

## Build & Test

Run anything related to SUAVE execution inside the `suave_runner` container, including tests, ROS launches, `colcon`, and direct `pytest` runs. The host machine is not assumed to have SUAVE or ROS dependencies installed. Use the container's default sourced workspace configuration; do not override `PYTHONPATH`, `ROS_LOG_DIR`, or similar ROS/Python environment variables unless the user explicitly asks.

Default container command pattern:

```bash
docker exec suave_runner bash -lc 'cd /home/ubuntu-user/suave_ws && source /opt/ros/humble/setup.bash && source install/setup.bash && <command>'
```

All build/test commands run from the **ROS workspace root inside the container** (`/home/ubuntu-user/suave_ws`), not from inside `src/suave`.

```bash
source /opt/ros/humble/setup.bash

# Install deps
rosdep install --from-paths src --ignore-src -r -y

# Build all
colcon build --symlink-install

# Low-memory build
colcon build --symlink-install --executor sequential --parallel-workers 1

# Build single package
colcon build --symlink-install --packages-select <package_name>

source install/setup.bash

# Test a package
colcon test --packages-select <package_name> --event-handlers console_direct+
colcon test-result --verbose

# Run Python tests directly (after sourcing)
python3 -m pytest -q <package>/test
```

Lint tests are pytest wrappers around `ament_flake8`, `ament_pep257`, and `ament_copyright`.

## Running SUAVE

Run SUAVE from inside the `suave_runner` container using the default workspace environment.

```bash
# Quick example
cd runner && ./example_run.sh

# ArduSub SITL (separate terminal)
sim_vehicle.py -L RATBeach -v ArduSub --model=JSON --console

# Simulation
ros2 launch suave simulation.launch.py x:=-17.0 y:=2.0

# Mission (default: no adaptation manager)
ros2 launch suave_missions mission.launch.py

# Mission with a specific manager
ros2 launch suave_missions mission.launch.py adaptation_manager:=bt result_filename:=measurement_1
# adaptation_manager values: none | metacontrol | random | bt

# Experiment runner (ROS2, config-file driven — preferred for campaigns)
ros2 launch suave_runner suave_runner.launch.py
# Config: suave_runner/config/runner_config.yml — controls experiments, disturbance timing, result_path

# Shell runner (simple positional args)
cd runner && ./runner.sh [true|false] [metacontrol|random|none|bt] [time|distance] <runs>
# headless_runner.sh is the same but uses screen instead of xfce4-terminal

# Results default to: ~/suave/results/
```

MAVROS default FCU URL: `udp://0.0.0.0:14550@14555` (avoids needing `sim_vehicle --out=...`).

## Docker

```bash
# Run GUI image
docker run -it --shm-size=512m -p 6901:6901 -e VNC_PW=password --security-opt seccomp=unconfined ghcr.io/kas-lab/suave:main

# Build all images locally
./build_docker_images.sh

# Build headless image (from repo root)
docker build -t suave-headless:dev -f docker/dockerfile-suave-headless .

# Syntax check
docker build --check -f docker/dockerfile-suave-headless .
docker build --check --build-arg BASE_IMAGE=kasm-jammy:dev -f docker/dockerfile-suave .
```

Dockerfiles are intentionally lowercase (`docker/dockerfile-*`). When checking `docker/dockerfile-suave` without a local base, GHCR may return `denied` — use `--build-arg BASE_IMAGE=kasm-jammy:dev` after building it locally.

**Image users and result paths:**
- GUI image (`suave:main`): user `kasm-user`, results at `/home/kasm-user/suave/results`
- Headless image (`suave-headless:main`): user `ubuntu-user`, results at `/home/ubuntu-user/suave/results`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kas-lab/suave](https://github.com/kas-lab/suave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

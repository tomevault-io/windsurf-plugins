---
trigger: always_on
description: SUAVE (Self-adaptive Underwater Autonomous Vehicle Exemplar) is a ROS 2 Humble exemplar for a single AUV pipeline-inspection mission: search for a pipeline, follow it, and inspect it. The repository separates the managed subsystem (vehicle mission functionality) from managing subsystems (adaptation logic), so adaptation managers can be swapped through standard ROS 2 interfaces.
---

# Repository Guidelines

## Project Overview

SUAVE (Self-adaptive Underwater Autonomous Vehicle Exemplar) is a ROS 2 Humble exemplar for a single AUV pipeline-inspection mission: search for a pipeline, follow it, and inspect it. The repository separates the managed subsystem (vehicle mission functionality) from managing subsystems (adaptation logic), so adaptation managers can be swapped through standard ROS 2 interfaces.

Runtime stack: ROS 2 Humble, Gazebo Harmonic, ArduSub/ArduPilot SITL, MAVROS, BehaviorTree.CPP, MROS2/Metacontrol, and Docker/Kasm.

## Project Structure & Module Organization

Core managed-system Python nodes live in `suave/suave/`, with launch files and sim config in `suave/launch/` and `suave/config/`. Monitoring nodes are in `suave_monitor/`, missions and mission configs in `suave_missions/`, metrics in `suave_metrics/`, auxiliary tools in `suave_tools/`, and experiment orchestration plus statistical analysis in `suave_runner/`. Managing subsystems are under `suave_managing/`, including `suave_none`, `suave_random`, `suave_metacontrol`, and the C++ BehaviorTree.CPP package `suave_bt`. Custom services are defined in `suave_msgs/srv/`. Tests are usually in each package's `test/` directory. Docker assets are in `docker/`, runner scripts in `runner/`, and documentation in `docs/source/`.

## Build, Test, and Development Commands

Run anything related to SUAVE execution inside the `suave_runner` container, including tests, ROS launches, `colcon`, and direct `pytest` runs. The host machine is not assumed to have SUAVE or ROS dependencies installed. Use the container's default sourced workspace configuration; do not override `PYTHONPATH`, `ROS_LOG_DIR`, or similar ROS/Python environment variables unless the user explicitly asks.

Default container command pattern:

```bash
docker exec suave_runner bash -lc 'cd /home/ubuntu-user/suave_ws && source /opt/ros/humble/setup.bash && source install/setup.bash && <command>'
```

Run ROS build and test commands from the ROS workspace root inside the container, `/home/ubuntu-user/suave_ws`, not from inside `src/suave`.

```bash
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
colcon build --symlink-install --executor sequential --parallel-workers 1
colcon build --symlink-install --packages-select <package_name>
source install/setup.bash
colcon test --packages-select <package_name> --event-handlers console_direct+
colcon test-result --verbose
python3 -m pytest -q <package>/test
```

After changing package data, launch files, setup metadata, or installed config files, rebuild with `colcon build --symlink-install` so installed resources are refreshed.

## Running SUAVE

Run SUAVE from inside the `suave_runner` container using the default workspace environment.

Use `cd runner && ./example_run.sh` for a full example. For manual runs:

```bash
# ArduSub SITL in a separate terminal
sim_vehicle.py -L RATBeach -v ArduSub --model=JSON --console

# Simulation
ros2 launch suave simulation.launch.py x:=-17.0 y:=2.0

# Mission, defaulting to no adaptation manager
ros2 launch suave_missions mission.launch.py

# Mission with a specific manager
ros2 launch suave_missions mission.launch.py adaptation_manager:=bt result_filename:=measurement_1
```

Valid `adaptation_manager` values are `none`, `metacontrol`, `random`, and `bt`. The preferred campaign runner is `ros2 launch suave_runner suave_runner.launch.py`; its config is `suave_runner/config/runner_config.yml` and results default to `~/suave/results/`. The shell runner is `cd runner && ./runner.sh [true|false] [metacontrol|random|none|bt] [time|distance] <runs>`, with `headless_runner.sh` using `screen` instead of `xfce4-terminal`.

MAVROS default FCU URL is `udp://0.0.0.0:14550@14555`, avoiding the need for `sim_vehicle --out=...`.

## Docker

```bash
docker run -it --shm-size=512m -p 6901:6901 -e VNC_PW=password --security-opt seccomp=unconfined ghcr.io/kas-lab/suave:main
./build_docker_images.sh
docker build -t suave-headless:dev -f docker/dockerfile-suave-headless .
docker build --check -f docker/dockerfile-suave-headless .
docker build --check --build-arg BASE_IMAGE=kasm-jammy:dev -f docker/dockerfile-suave .
```

Dockerfiles are intentionally lowercase as `docker/dockerfile-*`. When checking `docker/dockerfile-suave` without a local base, GHCR may return `denied`; use `--build-arg BASE_IMAGE=kasm-jammy:dev` after building the base locally. GUI image results are under `/home/kasm-user/suave/results`; headless image results are under `/home/ubuntu-user/suave/results`. The headless image has no `CMD`, so pass the runner command explicitly on `docker run`.

Version pins live in `docker/versions.env`, and Python package versions live in repository-root `requirements.txt`. `build_docker_images.sh` sources `versions.env` and forwards SHAs as build args.

## ROS Interfaces for Managing Subsystems


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kas-lab/suave](https://github.com/kas-lab/suave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

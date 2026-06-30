---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IR-SIM is an open-source, Python-based, lightweight robot simulator for navigation, control, and reinforcement learning. It uses YAML-driven configuration for defining robots, sensors, obstacles, and environments with built-in collision detection.

## Development Commands

```bash
# Install dependencies (using uv)
uv sync

# Run all tests
pytest

# Run tests with coverage
pytest --cov . --cov-report=html

# Run a single test file
pytest tests/test_kinematics.py

# Run a specific test
pytest tests/test_kinematics.py::test_name

# Linting and formatting
ruff check                    # Check for lint errors
ruff check --fix              # Auto-fix lint errors
ruff format                   # Format code

# Pre-commit hooks (install once)
pre-commit install

# Build documentation
cd docs && make html
```

## Architecture

### Entry Point
`irsim.make('config.yaml')` creates an environment from a YAML configuration file. The main simulation loop uses `env.step()`, `env.render()`, and `env.done()`.

### Core Components

**Environment Layer** (`irsim/env/`):
- `EnvBase` / `EnvBase3D`: Main environment classes handling simulation lifecycle
- `env_config.py`: YAML configuration parsing
- `env_plot.py` / `env_plot3d.py`: Matplotlib-based visualization (2D and 3D)
- `env_logger.py`: Environment logging

**World Layer** (`irsim/world/`):
- `World` / `World3D`: Core simulation state and collision detection (2D and 3D)
- `object_base.py`: Base class for all simulation objects (robots, obstacles)
- `object_factory.py`: Factory pattern for creating objects from YAML
- `object_group.py`: Grouping mechanism for coordinated multi-agent behaviors

**Robot Kinematics** (`irsim/world/robots/`):
- `robot_diff.py`: Differential drive
- `robot_omni.py`: Omnidirectional
- `robot_acker.py`: Ackermann steering (car-like)

**Obstacles** (`irsim/world/obstacles/`):
- `obstacle_static.py`: Static obstacles
- `obstacle_diff.py`: Differential drive dynamic obstacles
- `obstacle_omni.py`: Omnidirectional dynamic obstacles
- `obstacle_acker.py`: Ackermann steering dynamic obstacles

**Behaviors** (`irsim/lib/behavior/`):
- Registry-based system using decorators (`behavior_registry.py`)
- Individual behaviors registered by robot type + behavior name:
  - `diff`: `dash`, `rvo`, `sfm`
  - `omni`: `dash`, `rvo`, `sfm`
  - `omni_angular`: `dash`
  - `acker`: `dash`
- Group behaviors in `group_behavior.py` and `group_behavior_methods.py`:
  - `orca` (optimal reciprocal collision avoidance) - requires `pyrvo` package
- SFM algorithm implementation: `irsim/lib/algorithm/social_force_model.py` (anisotropic Moussaid-Helbing 2009 variant)

**Path Planners** (`irsim/lib/path_planners/`):
- `a_star.py`: A* grid-based path planning
- `rrt.py`: Rapidly-exploring Random Tree
- `rrt_star.py`: RRT* optimized path planning
- `informed_rrt_star.py`: Informed RRT* path planning
- `jps.py`: Jump Point Search (optimized A* variant)
- `probabilistic_road_map.py`: PRM path planning

**Sensors** (`irsim/world/sensors/`):
- `lidar2d.py`: 2D LiDAR simulation
- `fmcw_lidar2d.py`: Simplified 2D FMCW LiDAR with per-beam radial velocity
- `sensor_factory.py`: Factory for sensor instantiation

**Map** (`irsim/world/map/`):
- `obstacle_map.py`: Obstacle map representation
- `grid_map_generator_base.py`: Base grid map generator
- `image_map_generator.py`: Image-based map generation
- `perlin_map_generator.py`: Perlin noise procedural map generation
- `binary_map_generator_hm3d/`: HM3D binary map generator

**GUI** (`irsim/gui/`):
- `keyboard_control.py`: Keyboard-based robot control (requires `pynput`)
- `mouse_control.py`: Mouse-based interaction

### Key Patterns

- **YAML Configuration**: All scenarios defined in human-readable YAML files
- **Factory Pattern**: Objects created via `object_factory.py` from YAML specs
- **Registry Pattern**: Behaviors and sensors registered via decorators for extensibility
- **Geometry via Shapely**: Collision detection uses Shapely library (>=2.1.2)
- **Centralized RNG**: All randomness routes through `irsim.util.random.rng` (a proxy over `numpy.random.Generator`); call `set_seed(seed)` to make runs reproducible

### Directory Structure

```
irsim/                  # Main package
├── env/                # Environment and visualization (2D/3D)
├── world/              # Core simulation components
│   ├── robots/         # Robot kinematics (diff, omni, acker)
│   ├── obstacles/      # Obstacle types (static, dynamic)
│   ├── sensors/        # Sensor implementations (lidar2d, fmcw_lidar2d)
│   ├── map/            # Map generators (grid, image, perlin, hm3d)
│   └── description/    # Robot/vehicle visualization assets (PNG)
├── lib/                # Algorithms and behaviors
│   ├── behavior/       # Robot behaviors (dash, rvo, sfm, orca)
│   ├── algorithm/      # Core algorithms (kinematics, rvo, sfm, generation)
│   ├── path_planners/  # Path planning (A*, RRT, RRT*, Informed RRT*, JPS, PRM)
│   └── handler/        # Geometry and kinematics handlers
├── gui/                # Keyboard/mouse controls
├── util/               # Utility functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hanruihua/ir-sim](https://github.com/hanruihua/ir-sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

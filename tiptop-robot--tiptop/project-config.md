---
trigger: always_on
description: Generates 6-DOF grasps from point clouds. Installation:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TiPToP is a Task and Motion Planning (TAMP) system for robots with a modular architecture consisting of three sequential components:

1. **Perception Module** - Processes RGB images and natural language to build object-centric 3D representations using FoundationStereo (depth), Gemini Robotics-ER 1.5 (VLM), SAM-2 (segmentation), and M2T2 (grasp detection)
2. **Planning Module** - Uses cuTAMP, a GPU-parallelized TAMP solver that optimizes thousands of candidate pick-and-place plans in parallel
3. **Execution Module** - Executes trajectories using joint impedance control

More details: https://tiptop-robot.github.io/

## Development Commands

### Installation

We use [pixi](https://pixi.prefix.dev/) for environment and dependency management:

```bash
# Install all dependencies (PyTorch, cuRobo deps, etc.)
pixi install

# Install external dependencies (ZED SDK, cuRobo, cuTAMP)
pixi run setup-all
```

### Activating the Environment

All development and CLI commands must be run inside the pixi shell:

```bash
# Activate the pixi environment
pixi shell

# Now you can run all tiptop commands directly
```

### CLI Scripts

The package provides several CLI entry points (defined in `pyproject.toml`). First activate the environment with `pixi shell`, then run commands directly:

```bash
# Setup and calibration
compute-gripper-mask       # Gemini + SAM2 gripper detection

# Visualization
viz-calibration            # Rerun visualization of camera calibration
viz-gripper-cam            # View gripper camera feed
viz-scene                  # Visualize scene

# Robot control
go-home                    # Move robot to home configuration
go-to-capture              # Move robot to image capture configuration
gripper-open               # Open gripper
gripper-close              # Close gripper
```

### Documentation

Documentation tasks can be run directly with `pixi run`:

```bash
# Install Sphinx and doc dependencies
pixi run docs-install

# Build HTML documentation
pixi run docs-build

# Serve with live reload (for development)
pixi run docs-serve

# Clean build artifacts
pixi run docs-clean
```

The documentation is automatically built and deployed via ReadTheDocs using Python 3.11.

## Architecture Overview

### Package Structure

The `tiptop/` package is organized into focused modules:

- **`config/`** - Centralized configuration management using OmegaConf
  - `tiptop.yml` contains robot, camera, and perception service settings
  - `calibration_info.json` stores camera-to-end-effector transforms
  - `tiptop_cfg()` provides lazy-loaded config with CLI override support

- **`perception/`** - Perception pipeline with microservices architecture
  - `foundation_stereo.py` - Depth estimation via HTTP to remote server
  - `m2t2.py` - Grasp detection via HTTP to remote server
  - `sam2.py` - Object segmentation (local or remote)
  - `zed_camera.py` - ZED stereo camera interface
  - All services support both sync and async (`_async`) APIs

- **`scripts/`** - CLI entry points for setup, visualization, and robot control

- **`motion_planning.py`** - Motion planning using cuRobo's MotionGen

- **`warm_start.py`** - Initializes and warm-ups IK solver and MotionGen

- **`tiptop_demo.py`** - Main end-to-end demo pipeline

### Data Flow

The system follows this pipeline:

```
1. Image Capture
   └─> ZED camera (RGB-D stereo)

2. Perception (Async where possible)
   ├─> FoundationStereo (depth refinement)
   ├─> Depth-to-3D projection (point cloud)
   ├─> Gemini (object detection from RGB)
   ├─> SAM2 (object segmentation)
   └─> M2T2 (grasp generation from point cloud)

3. Task Planning
   ├─> Create TAMP environment (objects + surfaces)
   ├─> Parse instruction with Gemini
   └─> Run cuTAMP for motion plans

4. Execution
   ├─> IK solver for grasp poses
   ├─> MotionGen for collision-aware trajectories
   └─> Bamboo Franka client for robot control

5. Visualization
   └─> Rerun for real-time 3D visualization
```

### Key Integration Points

- **Bamboo Franka Controller** (`bamboo-franka-controller`) - Joint position queries and trajectory execution with impedance control
- **cuRobo** - GPU-accelerated motion planning and collision checking
- **cuTAMP** - Task-level planning with stream-based constraint solving (external dependency, not in `pyproject.toml`)
- **Rerun** - 3D visualization of robot state, sensor data, and plans
- **OmegaConf** - Configuration management with YAML files and CLI overrides

### Coordinate Frame Conventions

Transformation matrices follow the pattern `target_from_source`:
- `world_from_cam` - Transforms camera coordinates to world coordinates
- `obj_from_grasp` - Transforms grasp frame to object frame
- `gripper_from_cam` - Transforms camera frame to gripper frame

Calibration data in `calibration_info.json` stores 4x4 homogeneous transforms for camera extrinsics.

## Microservices Architecture

TiPToP uses a microservices-based architecture where external components run as separate HTTP servers:

### M2T2 (Grasp Detection)
Generates 6-DOF grasps from point clouds. Installation:

```bash
cd $TiPToP_DIR
git clone git@github.com:williamshen-nz/m2t2-private.git M2T2
cd M2T2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tiptop-robot/tiptop](https://github.com/tiptop-robot/tiptop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

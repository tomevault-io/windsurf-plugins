---
trigger: always_on
description: This file provides guidance to AI coding agents (OpenHands, Claude Code, etc.) when working with the AirStack repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (OpenHands, Claude Code, etc.) when working with the AirStack repository.

## Quick Start for AI Agents

**Project:** AirStack - Comprehensive autonomous aerial robotics stack developed by AirLab CMU

**Stack:** ROS 2 Jazzy | Docker-based development | Isaac Sim with Pegasus extension | Microsoft AirSim (legacy, UE4) | Field robotics

**Primary Goal:** Enable agents to understand the architecture, implement new algorithms/modules, and integrate them correctly into the layered autonomy stack.

## Repository Purpose

AirStack provides a complete end-to-end system for autonomous drone operations including:
- Modular autonomy stack (interface, sensors, perception, local planning, global planning, behavior)
- High-fidelity simulation environments (Isaac Sim with Pegasus extension, Microsoft AirSim legacy/UE4)
- Ground Control Station for mission planning and monitoring
- Multi-robot coordination capabilities
- Hardware deployment tools

The architecture is designed to allow easy swapping of algorithm modules (e.g., different planners, controllers, perception systems) through a standardized ROS 2 interface pattern.

## Repository Architecture

### High-Level Structure
```
AirStack/
├── robot/                    # Onboard autonomy stack (ROS 2 Jazzy)
│   └── ros_ws/src/          # Layered autonomy modules
│       ├── interface/       # Hardware interface & safety
│       ├── sensors/         # Sensor integration
│       ├── perception/      # State estimation & perception
│       ├── local/           # Local planning, world models, control
│       ├── global/          # Global planning & mapping
│       └── behavior/        # High-level mission execution
├── simulation/isaac-sim/    # Isaac Sim (Pegasus extension)
├── simulation/ms-airsim/       # AirSim (UE4 + PX4 SITL)
├── gcs/                     # Ground Control Station
├── common/                  # Shared packages & utilities
├── docs/                    # MkDocs documentation
├── mkdocs.yml               # MkDocs config file
├── tests/                   # System tests (pytest) + metrics reporting
├── .github/
│   ├── workflows/           # GitHub Actions CI (system-tests, docker-build, etc.)
│   └── orchestrator/        # OpenStack-backed ephemeral self-hosted runners
└── .agents/skills/          # Detailed workflow guides for agents
```

### Layered Autonomy Pattern

The autonomy stack follows a **layered architecture** where data flows through processing stages:

```
Sensors → Perception → World Models → Planners → Controllers → Interface → Hardware
```

Each layer has:
- **Module packages**: Individual algorithm implementations (e.g., `droan_local_planner`)
- **Bringup package**: Orchestrates layer launch with topic remapping (e.g., `local_bringup`)

**Key Insight:** Understanding "what connects to what" is critical. See [Integration Checklist](docs/robot/autonomy/integration_checklist.md) and [System Architecture](docs/robot/autonomy/system_architecture.md).

## Standard Topic Patterns

Modules communicate via ROS 2 topics. Common standard topics:

| Topic Pattern | Type | Purpose |
|--------------|------|---------|
| `/{robot_name}/odometry` | nav_msgs/Odometry | Robot state estimation |
| `/{robot_name}/global_plan` | nav_msgs/Path | Global waypoint path |
| `/{robot_name}/trajectory_controller/trajectory_override` | airstack_msgs/TrajectoryOverride | Direct trajectory commands |
| `/{robot_name}/trajectory_controller/trajectory_segment_to_add` | airstack_msgs/TrajectorySegment | Planned trajectory segment |
| `/{robot_name}/trajectory_controller/look_ahead` | geometry_msgs/PointStamped | Look-ahead point for planning |

**Note:** Topics are remapped in bringup launch files to connect modules. Input/output topics should be configurable via launch arguments.

See [Integration Checklist](docs/robot/autonomy/integration_checklist.md) for comprehensive topic conventions.

## Common Workflows (Skills)

For detailed step-by-step instructions, refer to the **`.agents/skills/`** directory:

| Skill | When to Use |
|-------|------------|
| [add-ros2-package](.agents/skills/add-ros2-package) | Creating a new algorithm module package |
| [add-task-executor](.agents/skills/add-task-executor) | Implementing a task executor as a ROS 2 action server |
| [integrate-module-into-layer](.agents/skills/integrate-module-into-layer) | Adding module to layer bringup |
| [write-launch-file](.agents/skills/write-launch-file) | Authoring ROS 2 launch files with AirStack conventions (ROBOT_NAME namespacing, topic remapping, allow_substs) |
| [write-isaac-sim-scene](.agents/skills/write-isaac-sim-scene) | Creating custom simulation scenes |
| [visualize-in-foxglove](.agents/skills/visualize-in-foxglove) | Adding topic visualization to Foxglove/GCS |
| [attach-gossip-payload](.agents/skills/attach-gossip-payload) | Broadcasting custom ROS messages to peers via PeerProfile gossip payloads |
| [debug-module](.agents/skills/debug-module) | Autonomous debugging of ROS 2 modules |
| [update-documentation](.agents/skills/update-documentation) | Documenting new modules and updating mkdocs |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [castacks/AirStack](https://github.com/castacks/AirStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->

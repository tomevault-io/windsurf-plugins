---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LingTu (灵途) is an autonomous navigation system for quadruped robots in outdoor/off-road environments.

- **Platform**: S100P (RDK X5, Nash BPU 128 TOPS, aarch64) | ROS2 Humble | Ubuntu 22.04
- **Languages**: Python (framework + semantic modules), C++ (SLAM/terrain/planner)
- **Architecture**: Module-First — Module is the only runtime unit, Blueprint is the only orchestration
- **Guideline**: `docs/MODULE_FIRST_GUIDELINE.md` — 8 rules for how code should be structured

## Quick Start

```bash
# Framework tests (no ROS2 needed, runs on any machine)
python -m pytest src/core/tests/ -q       # 1226 tests

# CLI with interactive REPL (profile-based, recommended)
python lingtu.py                          # interactive profile selector
python lingtu.py stub                     # no hardware, framework testing
python lingtu.py sim                      # MuJoCo simulation (full stack)
python lingtu.py dev                      # semantic pipeline, no C++ nodes
python lingtu.py s100p                    # real S100P robot (BPU + Kimi)
python lingtu.py explore                  # exploration, no pre-built map
python lingtu.py map                      # mapping mode (SLAM + save)
python lingtu.py --list                   # list all profiles

# Override any profile flag
python lingtu.py s100p --llm mock         # real robot but mock LLM
python lingtu.py s100p --daemon           # background daemon (S100P)
python lingtu.py stop                     # stop running daemon

# Composable factory API (each line = one functional stack)
from core.blueprint import autoconnect
from core.blueprints.stacks import *
system = autoconnect(
    driver("thunder", host="192.168.66.190"),   # L1 Robot + camera bridge
    slam("localizer"),                           # L1 SLAM / localization
    maps(),                                      # L2 OccupancyGrid + ESDF + ElevationMap
    perception("bpu"),                           # L3 Detector + Encoder + Reconstruction
    memory(),                                    # L3 SemanticMapper + Episodic + Tagged + VectorMemory
    planner("kimi"),                             # L4 SemanticPlanner + LLM + VisualServo
    navigation("astar"),                         # L5 Navigation + Autonomy chain
    safety(),                                    # L0 SafetyRing + Geofence
    gateway(5050),                               # L6 HTTP/WS/SSE + MCP + Teleop
).build()
system.start()
```

## Architecture — Module-First with Composable Stacks

Module is the only runtime unit. Blueprint composes Modules. Factory functions bundle related Modules into reusable stacks.

### Layer Hierarchy

```
L0  Safety       — SafetyRingModule + GeofenceManagerModule + CmdVelMux
L1  Hardware     — Driver + CameraBridge + SLAM (managed/bridge/localizer)
L2  Maps         — OccupancyGrid + ESDF + ElevationMap + Terrain + LocalPlanner + PathFollower
L3  Perception   — Detector + Encoder + Reconstruction + SemanticMapper + Episodic + Tagged + VectorMemory
L4  Decision     — SemanticPlanner + LLM + VisualServo (bbox tracking + person following)
L5  Planning     — NavigationModule (A*/PCT + WaypointTracker + mission FSM + goal safety)
L6  Interface    — Gateway + MCP + Teleop
```

High layers → low layers only. L5→L2 (waypoint→PathFollower) is command dispatch, not dependency.

### Composable Stack Factories (`src/core/blueprints/stacks/`)

| Factory | Returns | Modules |
|---------|---------|---------|
| `driver(robot)` | Blueprint | Driver + CameraBridge (auto-detect) |
| `slam(profile)` | Blueprint | SLAMModule or SlamBridgeModule |
| `maps()` | Blueprint | OccupancyGrid + ESDF + ElevationMap |
| `perception(det, enc)` | Blueprint | Detector + Encoder + Reconstruction |
| `memory(save_dir)` | Blueprint | SemanticMapper + Episodic + Tagged + VectorMemory |
| `planner(llm)` | Blueprint | SemanticPlanner + LLM + VisualServo |
| `navigation(planner)` | Blueprint | NavigationModule + Autonomy chain |
| `safety()` | Blueprint | SafetyRing + Geofence |
| `gateway(port)` | Blueprint | Gateway + MCP + Teleop |

### Pluggable Backends (via Registry)

| Module | Backends |
|--------|----------|
| Driver | `thunder` (gRPC→brainstem), `stub` (testing), `sim_mujoco`, `sim_ros2` |
| SLAM | `fastlio2`, `pointlio`, `localizer` (ICP on pre-built map), `bridge` (external ROS2) |
| Detector | `yoloe`, `yolo_world`, `bpu` (Nash hardware), `grounding_dino` |
| Encoder | `clip` (ViT-B/32), `mobileclip` (edge) |
| LLM | `kimi`, `openai`, `claude`, `qwen`, `mock` |
| Planner | `astar` (pure Python), `pct` (C++ ele_planner.so) |
| PathFollower | `nav_core` (C++ nanobind), `pure_pursuit`, `pid` |

All backends registered via `@register("category", "name")` in `core.registry`. Zero if/else.

### Profiles

| Profile | Driver | SLAM | Native | Semantic | Use Case |
|---------|--------|------|--------|----------|----------|
| `stub` | stub | none | no | no | Framework testing |
| `dev` | stub | none | no | yes | Semantic pipeline dev |
| `sim` | sim_ros2 | bridge | yes | yes | MuJoCo full simulation |
| `map` | sim_ros2 | fastlio2 | no | no | Build map with SLAM |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kitjesen/MapPilot](https://github.com/Kitjesen/MapPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

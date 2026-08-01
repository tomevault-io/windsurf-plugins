---
trigger: always_on
description: Provides a language-agnostic pub-sub and service-call layer using ZeroMQ + Protobuf.
---

# MVSim — AI Agent Context

> **Maintenance rule:** Any change to the project (new class, new module, new sensor, refactor, dependency change, etc.) must be reflected in this file to keep it accurate for future AI tools.

---

## Project overview

**MultiVehicle Simulator (MVSim)** is a lightweight, realistic 2.5D dynamics simulator for mobile robots and multi-agent research. It runs standalone, as a ROS 2 node, or embedded in C++/Python applications. Worlds are fully configured via XML files. Current version: 1.3.0.

- License: BSD 3-Clause
- Language: C++17 (minimum CMake 3.9)
- Repo: https://github.com/MRPT/mvsim

---

## Repository layout

```
mvsim/
├── modules/               # Core C++ library
│   ├── simulator/         # Main simulation engine (libmvsim)
│   ├── comms/             # ZMQ/Protobuf pub-sub communications layer
│   └── msgs/              # Protobuf message definitions (.proto files)
├── mvsim_node_src/        # ROS 2 node wrapper
├── mvsim-cli/             # Command-line tool (mvsim launch/topic/node/server)
├── mvsim-pid-tuner/       # GUI tool for tuning PID controllers
├── mvsim_tutorial/        # Demo world XML files + launch files + RViz configs
├── definitions/           # Reusable vehicle and sensor XML definitions
├── examples_cpp/          # C++ subscriber and service-caller examples
├── examples_python/       # Python API examples
├── docs/                  # Sphinx + RST documentation
├── cmake/                 # CMake helpers and version template
├── .clang-format          # Code style (Google base, Allman braces, 4-space indent, tabs)
└── .clang-tidy            # Linter config (bugprone-*, selected cppcoreguidelines-*)
```

---

## Module: `modules/simulator` — `libmvsim`

The simulation engine. Headers live in `modules/simulator/include/mvsim/`.

### Key classes

| Class | Header | Role |
|---|---|---|
| `World` | `World.h` | Central simulation container. Owns all vehicles, blocks, world elements, Box2D physics world, GUI window, ZMQ comms client. Split across `World.cpp`, `World_gui.cpp`, `World_load_xml.cpp`, `World_services.cpp`, `World_simul.cpp`, `World_walls.cpp`. |
| `Simulable` | `Simulable.h` | Base interface for anything that steps through simulation time. Provides `simul_pre_timestep()`, `simul_post_timestep()`, pose access with shared mutex. |
| `VisualObject` | `VisualObject.h` | Base interface for anything renderable in the OpenGL GUI. |
| `VehicleBase` | `VehicleBase.h` | Abstract vehicle. Inherits `VisualObject` and `Simulable`. Holds wheels, sensors, friction model, controller, CSV logger. Created via `ClassFactory`. |
| `Block` | `Block.h` | Static or dynamic rigid-body obstacle/object in the world. |
| `HumanActor` | `HumanActor.h` | Pedestrian/human agent with motion model. |
| `Wheel` | `Wheel.h` | Per-wheel physical state (torque, slip, contact). |
| `SensorBase` | `Sensors/SensorBase.h` | Base class for all sensors. |
| `WorldElementBase` | `WorldElements/WorldElementBase.h` | Base class for ground plane, grids, elevation maps, etc. |
| `ControllerBase` | `ControllerBase.h` | Abstract controller interface. |
| `FrictionBase` | `FrictionModels/FrictionBase.h` | Abstract friction model. |
| `ClassFactory` | `ClassFactory.h` | Template factory that maps XML type strings to C++ constructors. |
| `TParameterDefinitions` | `TParameterDefinitions.h` | Declares XML-readable parameters with `DECLARE_PARAMETER_*` macros. |
| `Shape2p5` | `Shape2p5.h` | 2.5D shape (2D footprint + height interval). |
| `RemoteResourcesManager` | `RemoteResourcesManager.h` | Downloads and caches remote assets (maps, meshes). |
| `CsvLogger` | `CsvLogger.h` | Per-vehicle time-series logging to CSV. |
| `PID_Controller` | `PID_Controller.h` | Generic discrete PID used by controllers. |
| `CollisionShapeCache` | `CollisionShapeCache.h` | Caches Box2D collision shapes for mesh reuse. |
| `PoseTrajectoryFollower` | `PoseTrajectoryFollower.h` | Standalone (no World/Box2D dependency) pure-pursuit tracker for the "exactly reproducible trajectories" feature: evaluates a time-parameterized `(t,x,y)` polyline and computes the `(vx,omega)` twist to follow it. Used by the `trajectory` controller class (differential and Ackermann). |

### Vehicle dynamics (`src/VehicleDynamics/`)

| File | Vehicle type |
|---|---|
| `VehicleDifferential.cpp` | Differential drive (TurtleBot, Jackal style) |
| `VehicleAckermann.cpp` | Ackermann (car-like) with kinematic/dynamic control |
| `VehicleAckermann_Drivetrain.cpp` | Ackermann + realistic mechanical differentials (open/Torsen, 2WD/4WD) |

Each vehicle type has companion `*_Controller*.cpp` files for its controllers (Raw torque, Twist PID, Ideal twist, Front-steer PID, and — for `differential`/`ackermann` only — `trajectory`, the exactly-reproducible-trajectory controller built on `PoseTrajectoryFollower`).

### Sensors (`src/Sensors/`)

| File | Sensor |
|---|---|
| `LaserScanner.cpp` | 2D LiDAR (generic + RPLidar A2) |
| `Lidar3D.cpp` | 3D LiDAR (Velodyne VLP-16, Ouster OS1, Hesai Helios-32) |
| `CameraSensor.cpp` | RGB pinhole camera |
| `DepthCameraSensor.cpp` | RGBD depth + color camera |
| `IMU.cpp` / `ImuNoiseModel.cpp` | IMU with Forster 2016 noise model |
| `GNSS.cpp` | GPS/GNSS with configurable noise |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MRPT/mvsim](https://github.com/MRPT/mvsim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

---
trigger: always_on
description: Intrinsic Core is an open-source, hardware-agnostic robotics platform providing deterministic real-time control, collision-free motion planning, 6-DOF perception, automated camera calibration, and local physics simulation, coupled natively to ROS 2.
---

# AGENTS.md: Agent Guidelines & Architecture for Intrinsic Core

## 1. Executive Summary & Core Mental Model

Intrinsic Core is an open-source, hardware-agnostic robotics platform providing deterministic real-time control, collision-free motion planning, 6-DOF perception, automated camera calibration, and local physics simulation, coupled natively to ROS 2.

Robotic applications in Intrinsic Core are structured around four fundamental primitives:

### The 4 Core Primitives
1. **Assets (`intrinsic_asset_instance`)**: Self-contained, deployable packages representing hardware devices (manipulators, grippers, cameras), persistent compute services (neural network inference, pose estimators), simulators (Gazebo), or physical data/CAD meshes (SceneObjects).
2. **Skills (`skill_interface.Skill`)**: Discrete, parameterized, strongly-typed units of executable behavior invoked by the Behavior Tree Executive (e.g., `move_robot`, `capture_images`, `calibrate_camera_to_robot`, `adder`).
3. **World / Scene**: The spatial knowledge engine maintaining coordinate transforms (`/tf`), kinematic trees, CAD geometry, and collision environments. Modified dynamically via `ObjectWorldUpdates`.
4. **Executive**: The orchestration engine executing Behavior Trees (BT) to govern sequencing, concurrency, fallback handling, blackboard data transfer, and error recovery.

### The Runtime Architecture
* **Local Cluster**: Intrinsic Core executes on a local single-node Kubernetes (`k3s`) cluster running directly on the workstation or Industrial PC (IPC).
* **Cluster Gateway**: All CLI tools (`inctl`), Python Solution Building Language (SBL) scripts, and client libraries connect to the cluster gateway at `localhost:17080`.
* **Zenoh Middleware Router**: High-throughput DDS/ROS 2 bridge and telemetry exchange operates on port `7447`.
* **Execution Modes**: Solutions run in either `sim` (Gazebo physics & simulated sensors) or `real` (physical hardware modules & real-time `PREEMPT_RT` kernel).

---

## 2. Core Agent Principles & Architecture Rules

When authoring, refactoring, or running Intrinsic Core solutions, adhere strictly to these engineering tenets:

### Rule 1: Skills Are Strictly Stateless
> [!IMPORTANT]
> The Intrinsic Skill runtime creates a **new instance of your Skill class for every single interaction** (`get_footprint()`, `preview()`, and `execute()`).
* **Never** load heavy neural network weights, open persistent network sockets, or store mutable state inside a Skill's `__init__()`. Doing so causes the runtime to re-read files from disk and reconstruct inference sessions on every invocation, causing severe latency spikes and memory churn.
* **Always** offload state, continuous background processes, and large ML inference models to a **persistent Service Asset** (such as a containerized gRPC microservice). Author your custom Skill as a thin, lightweight client wrapper whose sole responsibility is to forward inputs and receive predictions from that background Service.

### Rule 2: Strongly-Typed Protobuf Contracts
* Every Skill and Service parameter and result **must** be declared using Protocol Buffers (`.proto`).
* Never pass arbitrary untyped JSON blobs or dictionaries across the Executive boundary. Protobuf schemas ensure compile-time safety, cross-language compatibility (Python/C++), and serialization validation.

### Rule 3: Progressive Asset Discovery (Reuse Before Re-inventing)
* Before writing a custom asset from scratch, check existing capabilities in:
  - Core catalog: [https://github.com/intrinsic-ai/intrinsic-core](https://github.com/intrinsic-ai/intrinsic-core) (`intrinsic/resources/catalog/`)
  - Core skills: [`.agents/skills/`](.agents/skills/)
  - Installed cluster assets: `inctl asset list --address localhost:17080`
* Only create a new custom Asset or Skill if no existing primitive meets the task requirements.

### Rule 4: Simulation-First Verification & Robotic Safety
> [!CAUTION]
> **Physical Robot Safety**: A physical robot or real hardware workcell may be connected to the cluster. **Never command physical motion or execute actions on real hardware (`--operation_mode=real`) without explicit consent and confirmation from the user.** Unintended physical motion poses severe safety risks to personnel and equipment.
* **Simulate First**: Always develop, test, and validate changes hermetically against Gazebo simulation (`--operation_mode=sim`) before considering physical hardware execution.
* **Inspect Spatial Alignment**: Inspect the world state before and after execution to verify that robot base poses, tool frames, collision geometries, and target frames match expectations prior to execution.

### Rule 5: Code Style & Formatting (Google Style)
The repository strictly adheres to **Google Code Style** across all languages, configuration files, and build definitions. Formatters run as standalone CLI tools (or via pre-commit), not as Bazel targets:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intrinsic-ai/intrinsic-core](https://github.com/intrinsic-ai/intrinsic-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

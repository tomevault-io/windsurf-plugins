---
trigger: always_on
description: Overview of InstinctLab design principles, usage, and expansion guide.
---

# InstinctLab Overview

## Design Principle
InstinctLab is the environment side of Project Instinct, built on top of **Isaac Lab**. It is designed for:
- **Isolation**: Follows Isaac Lab's "own project" workflow to preserve developer progress separate from the core repository, while still depending on it.
- **Flexibility**: Can be run as an Omniverse extension.
- **Unified Ecosystem**: Integrates with `instinct_rl` and `instinct_onboard`.
- **Modular Environment**: Extends Isaac Lab's `ManagerBasedRLEnv` to support advanced features like multi-critic RL and comprehensive monitoring.

## Critical Components

### 1. InstinctRlEnv (`instinctlab.envs.InstinctRlEnv`)
- **Description**: The core environment class that extends `ManagerBasedRLEnv`.
- **Key Additions**:
  - **MultiRewardManager**: Replaces the default reward manager to support multiple reward groups (useful for multi-critic RL).
  - **MonitorManager**: A dedicated manager for logging simulation status and metrics to TensorBoard.

### 2. MultiRewardManager (`instinctlab.managers.MultiRewardManager`)
- **Usage**: Used when `cfg.rewards` is an instance of `MultiRewardCfg`.
- **Function**: Computes rewards separately for defined groups.
- **Configuration**:
  ```python
  @configclass
  class RewardGroupsCfg(MultiRewardCfg):
      rewards = RewardsCfg() # Standard group
      # rewards_group_1 = RewardsCfg() # Additional group for 2nd critic
  ```

### 3. Motion Reference (`instinctlab.motion_reference`)
- **Description**: Manages motion data (e.g., AMASS), shadowing commands, and tracking/imitation rewards.
- **Components**:
  - **MotionReferenceManager**: Handles loading and streaming motion data. It manages multiple `MotionBuffer`s.
  - **MotionBuffer**: The interface for different motion sources (datasets, generative models).
  - **Shadowing Commands**: Generates commands for the robot to follow.
  - **Rewards**: `*_tracking_*` (time-based) and `*_imitation_*` (frame-based) rewards.

### 4. Virtual Obstacles (`instinctlab.terrains.virtual_obstacle`)
- **Description**: Generates abstract geometric representations (e.g., edge cylinders) from terrain meshes.
- **Usage**: Registered with sensors (like `VolumePointsSensor`) to enable collision detection and penetration computation without explicit physics collision geometry.

### 5. Noisy Grouped Sensor Camera (`instinctlab.sensors.NoisyGroupedRayCasterCamera`)
- **Description**: Extends `GroupedRayCaster` to add configurable noise pipelines and history buffers.
- **Purpose**: Sim-to-real transfer by simulating sensor artifacts (depth noise, stereo noise, latency).

### 6. Mesh Spawning (`instinctlab.sim.spawners.MeshFileCfg`)
- **Description**: Spawns rigid objects from mesh files (OBJ, STL, FBX) with automatic USD conversion.
- **Config options**:
  - **`make_instanceable`** (from MeshConverterCfg): When True, geometry is instanced for memory savings. Default True.
  - **`apply_collision_props_at_spawn`**: When False (default), collision is baked during conversion; spawn-time modify is skipped (avoids warning on instanced geometry). When True, collision props are applied at spawn for per-instance override; requires `make_instanceable=False` (auto-resolved in `__post_init__`).
- **Behavior matrix**:
  | make_instanceable | apply_collision_props_at_spawn | Result |
  |-------------------|-------------------------------|--------|
  | True | False | Default: no warning, instancing on, collision from converter |
  | False | True | Spawn-time collision override works |
  | False | False | No instancing, collision from converter only |
  | True | True | Auto-resolved: make_instanceable set to False |
- **Example** (default, no warning):
  ```python
  MeshFileCfg(asset_path=..., mass_props=..., collision_props=...)
  ```
- **Example** (spawn-time override):
  ```python
  MeshFileCfg(asset_path=..., collision_props=..., apply_collision_props_at_spawn=True, make_instanceable=False)
  ```

## Motion Buffer Design & Usage

### Concept
The `MotionBuffer` (`instinctlab.motion_reference.motion_buffer.MotionBuffer`) serves as the interface for motion data sources. It acts similarly to a PyTorch Dataset but designed for simulation:
- **Environment Assignment**: Each buffer is assigned a subset of environments to manage.
- **Data Filling**: It fills `MotionReferenceData` buffers with motion frames (joint positions, base pose, etc.) for specific timestamps.
- **State Management**: Handles resetting and initializing motion states for its assigned environments.

### Key Classes
- **`MotionBuffer`**: Base class. Implement this to create new motion sources.
- **`MotionReferenceData`**: Dataclass holding motion frames (tensors for `joint_pos`, `base_pos_w`, `link_pos_w`, etc.).
- **`MotionReferenceState`**: Dataclass holding the initial state for resets.

### Expanding Motion Reference (Creating Custom Buffers)
To support new data formats or generative motion sources, inherit from `MotionBuffer` and implement:

1.  **`reset(self, env_ids, ...)`**:
    - Reset internal state for the specified environments.
    - Update `symmetric_augmentation_mask_buffer` if needed.
2.  **`fill_init_reference_state(self, env_ids, ...)`**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [project-instinct/InstinctLab](https://github.com/project-instinct/InstinctLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

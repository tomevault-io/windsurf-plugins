---
trigger: always_on
description: This repository provides **state estimation and sensor fusion** for the
---

# AGENTS.md - mola_state_estimation

## Overview

This repository provides **state estimation and sensor fusion** for the
[MOLA](https://docs.mola-slam.org/latest/) (Modular Optimization framework for
Localization and mApping) framework. It fuses odometry, IMU, GNSS and pose
measurements to estimate a vehicle's pose, velocity and orientation over time.
It also provides offline georeferencing of keyframe-based maps.

License: GNU GPL v3 (commercial options available upon request).

## Repository layout

```
mola_state_estimation/              <- ROS 2 metapackage (no code, just dependency grouping)
mola_state_estimation_simple/       <- Lightweight constant-velocity kinematic estimator
mola_state_estimation_smoother/     <- Advanced iSAM2 factor-graph smoother
mola_gtsam_factors/                 <- Reusable custom GTSAM factor library
mola_georeferencing/                <- Georeferencing SimpleMaps with GNSS/IMU
docs/                               <- Sphinx/RST documentation sources
scripts/                            <- clang-format helpers
params/                             <- (inside smoother) default YAML configuration
```

## Sub-packages

### 1. `mola_state_estimation_simple`

A fast, minimal state estimator using a constant-velocity kinematic model with
incremental velocity extrapolation. Good enough for LiDAR-only odometry (LO/LIO)
on most automotive datasets.

| Item | Path |
|------|------|
| Main class | `mola_state_estimation_simple/include/.../StateEstimationSimple.h` |
| Parameters | `mola_state_estimation_simple/include/.../Parameters.h` |
| Implementation | `mola_state_estimation_simple/src/StateEstimationSimple.cpp` |
| Test | `mola_state_estimation_simple/tests/test-state-estimation-simple.cpp` |

Key traits:
- Inherits `mola::NavStateFilter` (abstract interface defined in `mola_kernel`).
- Fuses pose, odometry, IMU and twist; ignores GNSS.
- Not frame-aware (ignores `frame_id`).
- Supports optional planar-motion enforcement (`enforce_planar_motion`).

### 2. `mola_state_estimation_smoother`

An advanced sliding-window factor-graph smoother using GTSAM's iSAM2 for
incremental optimization. The primary estimator for multi-sensor fusion.

| Item | Path |
|------|------|
| Main class | `mola_state_estimation_smoother/include/.../StateEstimationSmoother.h` |
| Parameters | `mola_state_estimation_smoother/include/.../Parameters.h` |
| Implementation | `mola_state_estimation_smoother/src/StateEstimationSmoother.cpp` |
| Default config | `mola_state_estimation_smoother/params/state-estimation-smoother.yaml` |
| ROS 2 launch | `mola_state_estimation_smoother/ros2-launchs/ros2-state-estimator.launch.py` |
| MOLA-CLI launch | `mola_state_estimation_smoother/mola-cli-launchs/state_estimator_ros2.yaml` |
| CLI app | `mola_state_estimation_smoother/apps/mola-navstate-cli.cpp` |
| Tests (6) | `mola_state_estimation_smoother/tests/test-*.cpp` |
| Integration tests | `mola_state_estimation_smoother/test/integration/test_*.py` |

Key traits:
- Inherits `mola::NavStateFilter`, `mola::LocalizationSourceBase`,
  `mola::MapSourceBase`.
- Uses Pimpl pattern (`GtsamImpl`) to hide GTSAM details.
- Sliding time window of keyframes (default 2.5 s).
- Two kinematic models: `ConstantVelocity` and `Tricycle` (Ackermann).
- Multi-frame-aware: tracks multiple odometry sources by `frame_id`.
- Optional ENU-to-map georeferencing from GNSS.
- Thread-safe (`std::recursive_mutex`).
- Configuration via YAML with `${ENV_VAR|default}` substitution.

Sensor inputs:
- `fuse_pose()` - localization / LiDAR odometry poses
- `fuse_odometry()` - wheel odometry with uncertainty
- `fuse_imu()` - gravity alignment, angular velocity, attitude
- `fuse_gnss()` - GPS in ENU coordinates
- `fuse_twist()` - direct velocity measurements

### 3. `mola_gtsam_factors`

Reusable GTSAM factor classes for state estimation and georeferencing.

| Factor class | Purpose |
|--------------|---------|
| `FactorAngularVelocityIntegration` | Gyroscope-based rotation integration |
| `FactorConstLocalVelocity` | Constant-velocity prior in body frame |
| `FactorGnssEnu` | GNSS position measurement in ENU |
| `FactorGnssMapEnu` | GNSS with explicit ENU-to-map transform |
| `FactorTrapezoidalIntegrator` | Trapezoidal velocity integration |
| `FactorTricycleKinematic` | Ackermann / tricycle steering kinematics |
| `MeasuredGravityFactor` | Gravity-vector leveling from accelerometer |
| `Pose3RotationFactor` | Rotation-only constraint (decoupled from translation) |

Headers are in `mola_gtsam_factors/include/mola_gtsam_factors/`.
Implementations in `mola_gtsam_factors/src/`.
Most factors derive from `gtsam::ExpressionFactorN`; `Pose3RotationFactor`
derives from `gtsam::NonlinearFactor`.

### 4. `mola_georeferencing`

Offline georeferencing of MOLA SimpleMaps using GNSS and IMU observations.

| Item | Path |
|------|------|
| Library API | `mola_georeferencing/include/.../simplemap_georeference.h` |
| Implementation | `mola_georeferencing/src/simplemap_georeference.cpp` |
| CLI: georeference a simplemap | `mola_georeferencing/apps/mola-sm-georeferencing-cli.cpp` |
| CLI: georeference a trajectory | `mola_georeferencing/apps/mola-trajectory-georef-cli.cpp` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MOLAorg/mola_state_estimation](https://github.com/MOLAorg/mola_state_estimation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->

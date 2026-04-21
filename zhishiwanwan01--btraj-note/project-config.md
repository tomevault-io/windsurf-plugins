---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is `btrajSim_2025-08`, a ROS-based UAV trajectory planning simulation framework called Btraj. It generates safe, dynamically feasible trajectories in unknown environments using a two-stage approach: front-end path finding (Fast Marching* or A*) and back-end trajectory optimization with Bezier curves.

The system implements the algorithm from "Online Safe Trajectory Generation For Quadrotors Using Fast Marching Method and Bernstein Basis Polynomial" (Fei Gao et al., ICRA 2018).

## Build System & Dependencies
- **ROS Framework**: ROS Melodic (Ubuntu 18.04) - updated from Kinetic
- **Build System**: CMake with `catkin_make`
- **Key Dependencies**:
  - libarmadillo-dev (linear algebra library)
  - PCL (Point Cloud Library)
  - Eigen3
  - Mosek (QP solver - requires academic license)
  - fast_methods (Fast Marching implementation)
  - sdf_tools (Euclidean distance field computation)

## Development Commands

### Docker Environment (Recommended)
```bash
# From repository root - Build and start ROS Melodic container
./build.sh  # Auto-detects Windows IP for X11 forwarding via SSH

# Enter running container for development
./start.sh

# Inside container - build workspace
cd /root/catkin_ws
catkin_make
source devel/setup.bash
```

**Important**: `build.sh` automatically detects the Windows client IP address when connecting via SSH to configure X11 display forwarding. The `DISPLAY_IP` environment variable is set to `<detected-windows-ip>:0.0` and passed to docker-compose.

### Native Build (Without Docker)
```bash
# Install dependencies
sudo apt-get install libarmadillo-dev ros-melodic-desktop-full

# Build from catkin workspace root (NOT repository root)
cd catkin_ws
catkin_make
source devel/setup.bash
```

### Running Simulation
```bash
# Launch main simulation (includes rviz visualization)
roslaunch bezier_planer simulation.launch

# Use 3D Nav Goal tool in rviz to set waypoints:
# - Click tool, press left mouse button on position
# - Drag up/down while holding left button for height
# - Release to send target
```

### Configuration
- Set path finding algorithm in launch file: `is_use_fm` (true=Fast Marching*, false=A*)
- Map parameters, velocity/acceleration limits configurable in `simulation.launch`

## Architecture

### System Data Flow
The planning pipeline follows this sequence:
1. **User Input** → RViz 3D Nav Goal → `waypoint_generator` → publishes waypoints
2. **Sensing** → `random_forest_sensing` → publishes point cloud map based on sensor range
3. **Odometry** → `odom_generator` → simulates robot state feedback
4. **Planning** → `b_traj_node` receives waypoints + map + odometry:
   - Front-end: Generates collision-free path (FM* or A*)
   - Back-end: Optimizes Bezier trajectory within flight corridors
   - Publishes `PolynomialTrajectory` (Bezier coefficients)
5. **Execution** → `b_traj_server` → converts coefficients to real-time position commands
6. **Control Loop** → position commands → `odom_generator` → closes simulation loop

### Core Classes (catkin_ws/src/Btraj/include/)
- **TrajectoryGenerator** (`trajectory_generator.h`): QP solver interface using Mosek. Main function `BezierPloyCoeffGeneration()` takes flight corridors, constraints (vel/acc limits), and MQM matrices to generate optimal Bezier coefficients.
- **Bernstein** (`bezier_base.h`): Pre-computes basis transformation matrices (Bernstein↔Monomial), cost matrices (MQM), and combinatorial constants. Must be initialized before trajectory generation.
- **gridPathFinder** (`a_star.h`): A* implementation on 3D grid using multimap priority queue. Uses Manhattan/Euclidean/Diagonal heuristics.

### ROS Node Architecture (from simulation.launch)
The `bezier_planer` package compiles to 4 executables (see CMakeLists.txt):
- **b_traj_node** (b_traj_node.cpp): Main planner integrating path finding + trajectory optimization
  - Subscribes: `/waypoint_generator/waypoints`, `/odom/fake_odom`, `/random_forest_sensing/random_forest`
  - Publishes: `/position_cmd` (PositionCommand), `/b_traj_node/trajectory` (PolynomialTrajectory)
- **b_traj_server** (traj_server.cpp): Trajectory execution - converts polynomial coefficients to position commands
- **odom_generator** (odom_generator.cpp): Simulates odometry from position commands
- **random_forest_sensing** (random_forest_sensing.cpp): Generates random obstacle environment + sensor simulation

**Supporting packages** (catkin_ws/src/plan_utils/):
- `waypoint_generator`: Converts RViz goals to waypoint messages
- `odom_visualization`: Displays robot state in RViz
- `rviz_plugins`: Custom 3D Nav Goal tool (drag for height)
- `quadrotor_msgs`: Message definitions (PolynomialTrajectory, PositionCommand)

## Key Implementation Details

### b_traj_node.cpp Callback Structure
Three main callbacks drive the planning system:
- `rcvOdometryCallbck()`: Updates `_start_pt`, `_start_vel`, `_start_acc` from odometry
- `rcvPointCloudCallBack()`: Inflates obstacles into collision map, triggers replanning if `checkExecTraj()` detects collision
- `rcvWaypointsCallback()`: Receives goal, calls `trajPlanning()` to generate trajectory


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhishiwanwan01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

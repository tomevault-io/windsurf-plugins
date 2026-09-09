---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Guidelines for Claude

- **Action**: Do not jump into implementation or change files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action.
- **Code review**: Keep code reviews concise. Focus only on problems; skip complimenting the user for following the guidelines.

## Project Overview

Autoware Mini is a minimalistic Python-based autonomous vehicle software stack built on ROS 1 (Noetic). It's designed for teaching and research, validated with real-world deployment on a Lexus vehicle in Tartu, Estonia.

## Repository

- **GitLab**: `autonomous-driving-lab/autoware_mini` (project ID: 2286) at gitlab.cs.ut.ee - main branch: `main`
- **GitHub mirror**: `UT-ADL/autoware_mini` - main branch: `release`
- **Merge requests**: Unless specified otherwise, create merge requests for merging into `main` branch (GitLab)

## Releasing to GitHub

Public releases go to the GitHub mirror (`upstream` remote, `UT-ADL/autoware_mini`) on the
`release` branch. The `release` branch is a **squashed, disjoint history** from GitLab `main`: each
"Release X" is a single commit whose tree equals the then-current `main`, with the previous release
tip as its only parent. This keeps the public repo small. Releases use a simple sequential scheme
(`v0.1`, `v0.2`, …), independent of `package.xml`'s version.

To cut a release:

```bash
git fetch upstream                              # get the current release tip (may have direct PRs)
git checkout release && git reset --hard upstream/release
COMMIT=$(git commit-tree 'main^{tree}' -p HEAD -m "Release X.Y")
git reset --hard "$COMMIT"
git diff --stat main                            # GATE: must be EMPTY (release tree == main tree)
git tag vX.Y
git push upstream release && git push upstream vX.Y
git checkout main
gh release create vX.Y --repo UT-ADL/autoware_mini --title "Release X.Y" --notes-file <notes>
```

- **Build the commit on the current `upstream/release` tip**, not the local `release` branch (which
  is often stale). The empty `git diff --stat main` is the critical correctness gate.
- **Watch for release-branch-only commits**: community PRs are sometimes merged directly to the
  GitHub `release` branch and are absent from `main`. A raw squash of `main` reverts them. Fix such
  changes in `main` first (commit + push to GitLab), then squash — do not patch only the release commit.
- **Release notes**: group into `## Major changes` and `## Minor improvements`, one concise sentence
  each, ~10 total. Keep terse; don't name the app/medium unless it matters (e.g. monitoring is
  RViz-based — describe what it monitors, not "web dashboard"). Derive notes from the merged feature
  branches in `<prev-release-source>..main` (`git log --merges`).

## Build Commands

```bash
# Build the workspace (run from ~/autoware_mini_ws)
catkin build

# Release build for optimized performance
catkin build --cmake-args -DCMAKE_BUILD_TYPE=Release

# Source the workspace after building
source devel/setup.bash
```

## Running the System

```bash
# Lightweight planner simulation (no GPU required)
roslaunch autoware_mini start_sim.launch

# With real-time traffic lights from Tartu
roslaunch autoware_mini start_sim.launch tfl_detector:=mqtt

# Bag playback (recorded sensor data)
roslaunch autoware_mini start_bag.launch
roslaunch autoware_mini start_bag.launch detector:=lidar_sfa  # GPU detector

# CARLA simulation (requires CARLA_ROOT set)
roslaunch autoware_mini start_carla.launch

# Real Lexus vehicle
roslaunch autoware_mini start_lexus.launch
```

Common launch arguments: `detector:=` (lidar_cluster, lidar_sfa, radar, lidar_cluster_radar_fusion), `tfl_detector:=` (none, mqtt, camera, yolo)

### Restarting the System

If ROS processes are left hanging after a crash or interrupted launch:

```bash
# Kill all existing ROS nodes
rosnode kill -a

# Kill the rosmaster
pkill roscore

# Verify rosmaster is not running (should show error)
rostopic list
```

### Running on Neuron Server (neuron.hpc.ut.ee)

1. Launch scripts without RViz (no display available):
   - For launch files: `launch_rviz:=false`
   - For bash scripts: `--no_rviz`

2. Bag file locations:
   - `/data/bag_cache` - Locally cached bag files
   - `/data/Bolt/bagfiles` - All bag files ever recorded

## Architecture

The stack follows a modular pipeline:

```
SENSORS → LOCALIZATION → PERCEPTION → GLOBAL PLANNING → LOCAL PLANNING → CONTROL → VEHICLE
```

### Key Modules (all in nodes/)

- **localization/** - Vehicle position via GNSS (Novatel OEM7) or simulation ground truth
- **perception/** - Object detection (lidar_cluster, lidar_sfa neural network, radar), tracking (EMA), prediction, traffic lights
- **planning/global/** - Route planning using Lanelet2 maps, lane changes, waypoint management
- **planning/local/** - Trajectory generation with 17+ rule-based safety checkers (collision, traffic lights, crosswalks, etc.)
- **control/** - Path following via Pure Pursuit or Stanley controllers

### Python Library (src/autoware_mini/)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UT-ADL/autoware_mini](https://github.com/UT-ADL/autoware_mini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

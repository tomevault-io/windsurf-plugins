---
trigger: always_on
description: Learn ROS2 concepts with a simulated robotic arm + gripper + camera → eventually natural language control ("take the blue cube, put it in the basket"). Simulation-only for next 6+ months.
---

# ROS2 Robotic Arm Project

## Goal
Learn ROS2 concepts with a simulated robotic arm + gripper + camera → eventually natural language control ("take the blue cube, put it in the basket"). Simulation-only for next 6+ months.

## Instructions for Claude

**Documentation style — focus on WHY, not HOW:**
- The code is the source of truth for implementation details. Don't describe how things are implemented — document **decisions, constraints, and lessons learned**.
- No code examples in CLAUDE.md. Use `--help` or read the source.
- Keep CLAUDE.md **under 150 lines**. Move session notes to `doc/CHANGELOG.md` after each session.

**Always update this file** when you:
- Make an architectural or design decision
- Fix a non-obvious bug — document root cause and lesson
- Discover a constraint or ROS2/MoveIt quirk

**Also read and update these `doc/` companion files**:
- [`doc/MEMORY.md`](doc/MEMORY.md) — compact patterns and constraints
- [`doc/architecture.md`](doc/architecture.md) — node diagrams, topics, design decisions
- [`doc/CHANGELOG.md`](doc/CHANGELOG.md) — historical session notes

---

## Quick Reference

**What this is**: ROS2 Jazzy simulation of AR4 6-DOF arm + gripper. Phase 4.5 complete — AR4 switch with pick-and-place working, camera + vision pipeline, 44 integration tests (44/44 passing). No Gazebo, no LLM.

**Quick start**: `./run.sh sim` → launches RViz + MoveIt | `./run.sh tests` → runs integration tests

**Running nodes** (after `./run.sh sim`):
- `motion_server` — persistent action server, holds `ArmController` in memory, subscribes to `/detected_objects`
- `scene_manager` — one-shot, spawns objects from `objects.yaml` into MoveIt
- `move_group` — MoveIt2 planning server
- `arm_controller` / `gripper_controller` / `joint_state_broadcaster` — ros2_control
- `robot_state_publisher` + `rviz2`
- `camera_node` — synthetic top-down camera (Phase 4, working)
- `vision_node` — HSV color detection from pixels (Phase 4, working)

**Entry points**:
- CLI: `ros2 run robotic_arm_bringup arm <command> [args]` (thin action client)
- Actions: `/pick`, `/place`, `/move_to`, `/home`, `/reset`, `/open_gripper`, `/close_gripper`
- Services: `/list_objects`, `/move_object`
- Python: `from robotic_arm_bringup.arm_controller import ArmController`

**Key files**:
- Robot model: `src/robotic_arm_description/urdf/robotic_arm.urdf.xacro`
- Motion library: `src/robotic_arm_bringup/robotic_arm_bringup/arm_controller.py`
- Action server: `src/robotic_arm_bringup/robotic_arm_bringup/motion_server.py`
- CLI client: `src/robotic_arm_bringup/robotic_arm_bringup/arm_cli.py`
- Action/service definitions: `src/robotic_arm_interfaces/`
- Vision messages: `src/robotic_arm_interfaces/msg/DetectedObject.msg`, `DetectedObjects.msg`
- Object definitions: `src/robotic_arm_bringup/config/objects.yaml`
- Scene setup: `src/robotic_arm_bringup/robotic_arm_bringup/scene_manager.py`
- MoveIt config: `src/robotic_arm_moveit_config/`
- Synthetic camera: `src/robotic_arm_perception/robotic_arm_perception/camera_node.py`
- Vision detection: `src/robotic_arm_perception/robotic_arm_perception/vision_node.py`
- Launch: `src/robotic_arm_bringup/launch/arm_system.launch.py`

---

## Critical Constraints

These have caused bugs. Always remember them.

| Constraint | What breaks if ignored |
|------------|------------------------|
| Always set `start_state` on every MoveGroup plan | Second+ moves plan from home (0,0,0) → wrong trajectories |
| `AttachedCollisionObject` persists across CLI calls | Cube stays glued to arm across commands |
| Detach object before every `return False` in `place()` | Object stays attached after failed place |
| After `_detach_object`, call `_remove_object_from_scene` | MoveIt re-adds object at arm position → START_STATE_IN_COLLISION |
| Always re-add objects with `ObjectColor` after scene removal | Objects reappear green (MoveIt default) |
| MoveIt2 ACM diffs are NOT diffs — any non-empty ACM replaces the whole matrix | Must fetch current ACM, merge, then apply. Otherwise SRDF self-collision allowances wiped → OMPL rejects all states |
| Attach cube with `touch_links` BEFORE `close_gripper()` | Error 99999 (per-plan ACM doesn't work for gripper group) |
| Use `/apply_planning_scene` service (not topic) for attach/detach | `time.sleep()` doesn't spin executor → race condition |
| Mimic joints: no command interface in ros2_control | ros2_control crash on startup |
| Never call `spin_until_future_complete` on a node with an active executor | Deadlock — use `_wait_for_future()` instead |
| Never call `rclpy.spin_once(self)` on a node spun by `rclpy.spin()` | Executor deadlock — use `time.sleep()` polling instead |
| Never block inside a timer/service callback with `time.sleep()` polling | Blocks executor thread → service response callback can't fire → future never completes |
| MoveIt `CollisionObject.primitive_poses` are in object-local frame | World position is in `co.pose`, not `co.primitive_poses[0]` — reading primitive_poses gives (0,0,0) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daniel-iacob/ros2_robotic_arm](https://github.com/daniel-iacob/ros2_robotic_arm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->

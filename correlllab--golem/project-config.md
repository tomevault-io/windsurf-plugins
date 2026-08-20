---
trigger: always_on
description: Everything below was verified against the code on 2026-07-12. Where older docs or
---

# CLAUDE.md — GOLEM developer guide for coding agents

Everything below was verified against the code on 2026-07-12. Where older docs or
script comments disagree with the code, the code wins (known stale spots are
flagged inline).

## 1. Project overview

GOLEM (Generalized Open Layered Embodied Modules) is the software stack for a
Unitree H1-2 humanoid: MuJoCo/RoboCasa and Isaac Lab simulators, a ROS 2 Humble
workspace
(drivers, upper-body IK control, lower-body MPC/RL locomotion, safety layer,
perception model servers, and an LLM-callable skills layer), all running in
separate Docker containers that share one DDS domain. The same `rt/lowcmd` /
`rt/lowstate` Unitree DDS wire format is used against the sims and the real
robot, so code moves between them unchanged.

## 2. Repo layout

| Path | What it is |
|---|---|
| `docker/` | x86 setup — the fully-featured, preferred platform (Dockerfiles, `docker-compose.yml`, `scripts/`) |
| `docker/mac/` | self-contained Apple-Silicon (arm64, CPU-only) port — limited: no Isaac, no ML/vision stack, FastDDS instead of CycloneDDS |
| `core_ws/` | the ROS 2 workspace — where the bulk of development happens (see §4) |
| `h1_robocasa/` | RoboCasa/MuJoCo simulator entry point (`h12_mujoco.py`) and its ROS/DDS bridges |
| `CL_isaaclab_sim/` | Isaac Lab simulator (`sim_main.py`, DDS bridges, tasks) |
| `CL_Assets/` | URDF meshes, MuJoCo XML, Isaac USD (Git-LFS) |
| `mujoco_mpc/` | MuJoCo-MPC fork submodule (built standalone, outside colcon — see §8) |
| `unitree_sdk2_python/` | vendored Unitree DDS SDK (Python) |
| `tools/` | standalone debug tools (ROS MCP server for Claude Code) |
| `docs/` | `NAVIGATION_DEMO.md` (mac SLAM/nav2/frontier demo), `ROS_MCP_DEBUG.md` |
| `docker/BUILD.md` | deep dive on the image/build system — read it before touching any Dockerfile |

Most of the tree is **git submodules** (`git submodule update --init --recursive`
is mandatory) and large assets are **Git-LFS** (`git lfs install`).

## 3. Containers: always use the provided scripts

Never `docker run` the images by hand — the scripts wire up `docker/.env`, DDS
domain safety checks, bind-mount pre-creation, and stable container names.
They can be invoked from any directory.

### x86 (preferred platform)

| Script | Verified arguments | Notes |
|---|---|---|
| `docker/scripts/docker_build.sh [profile ...]` | `isaac`, `robocasa`, `ros` (no args = all three) | `golem_base` is built first automatically for `robocasa`/`ros`; `isaac` is self-contained. Pins the MJPC build to the `mujoco_mpc` submodule SHA (`MJPC_REF`). |
| `docker/scripts/docker_run.sh <profile> [cmd...]` | `isaac`, `robocasa`, `ros`; then optionally `bash` (shell instead of default launcher) or any command; a leading `-flag` (e.g. `--headless`) is forwarded to the default launcher | Container names: `golem_ros`, `golem_sim_robocasa`, `golem_sim_isaac`. |

Default launchers (compose `command`): `docker/scripts/launch_isaac.sh`,
`launch_robocasa.sh`, `launch_ros.sh`. The `ros` launcher colcon-builds
`core_ws` (if stale) and **drops to a shell — it does not launch bringup**.

### mac (`docker/mac`, limited)

| Script | Verified arguments | Notes |
|---|---|---|
| `docker/mac/scripts/docker_build_mac.sh [service ...]` | `robocasa`, `ros` (no args = both) | **No `isaac`** — Isaac Sim needs an NVIDIA GPU. arm64 base built from `docker/mac/BaseDockerfile.arm64`. |
| `docker/mac/scripts/docker_run_mac.sh <service> [cmd...]` | `robocasa`, `ros` | Starts ONE service. For the paired sim prefer `docker compose -f docker/mac/docker-compose.yml up` so both start together (RoboCasa running alone for a few seconds lets the robot collapse — motor command timeout). |

Mac feature toggles are env vars read by the compose file: `GOLEM_DISPLAY=vnc`
(MuJoCo viewer → noVNC :6080), `GOLEM_RVIZ=vnc` (RViz → :6081), `GOLEM_LOWERBODY=fame|walk|switch`,
`GOLEM_SLAM=1`, `GOLEM_NAV2=1`, `GOLEM_SIM_ODOM=1`, `GOLEM_CAMERAS=0`,
`GOLEM_SPAWN_BACKOFF=<m>`, `GOLEM_CMD_TIMEOUT=<sim-s>`, `GOLEM_ROS_MCP=1`.
GUIs stream over noVNC via `docker/mac/scripts/mac_vnc_tunnel.sh` (no XQuartz).

### DDS domain safety (both platforms)

`ROS_DOMAIN_ID=0` is the **real robot's command bus**. The run scripts refuse to
start a sim on it (the x86 `ros` profile asks for interactive confirmation; mac
always rejects). Unset/empty defaults to `1` (the sim domain). Set it in
`docker/.env` (copy from `docker/.env.example`; also holds `GEMINI_API_KEY`,
which the `ros` container needs for the vision servers and `h12_skills`).

## 4. `core_ws/src` packages

19 ROS packages in 16 directories. Note two name mismatches: dir `FAST_LIO` →
package `fast_lio`; dir `unitree_ros2` is not itself a package (it nests
`unitree_api`, `unitree_go`, `unitree_hg`, `unitree_ros2_example`).

| Directory | Role | What it does | Key entry points |
|---|---|---|---|
| `h1_bringup` | **bringup** | Launch-only package that starts the whole robot (sim or real). See §6. | 7 launch files, `config/*.yaml` |
| `h12_ros2_controller` | controller (upper body) | Pinocchio-based arm IK: `/frame_task` + `/dual_arm` action servers, joint state publisher, hand controller | `frame_task_server`, `dual_arm_server`, `joint_state_publisher`, `hand_controller_node` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [correlllab/GOLEM](https://github.com/correlllab/GOLEM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->

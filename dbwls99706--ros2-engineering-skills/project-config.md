---
trigger: always_on
description: >
---


# ROS 2 Engineering Skills

> **Single responsibility:** This skill is an **API reference & code template guide**
> for ROS 2 development. It tells you *how to use ROS 2 APIs correctly* and
> *what mistakes to avoid*. It does NOT do CI/CD orchestration, incident response,
> data analysis, or deployment automation — those are separate skill categories.

A progressive-disclosure skill for ROS 2 development — from first workspace to
production fleet deployment. Each section below gives you the essential decision
framework; detailed patterns, code templates, and anti-patterns live in the
`references/` directory. Read the relevant reference file before writing code.

## How to use this skill

**Progressive disclosure — do NOT read everything at once.**
This skill is structured in layers. Only load what you need for the current task:

1. **This file (SKILL.md)** — always loaded. Contains decision routing, core
   principles, pitfalls, and anti-patterns. Sufficient for answering quick
   questions and making architectural decisions.
2. **`references/*.md`** — load on demand. Use the Decision Router below to
   pick the 1–2 files relevant to the user's current task. Do NOT read all 20
   reference files — that wastes context and causes confusion.
3. **`scripts/`** — run only when the user needs code generation, QoS checking,
   or launch validation. These are tools, not reading material.

**Steps:**

1. If `.skill-runs.log` exists in the workspace, read the last few lines to
   understand what was done and what issues occurred in previous sessions.
2. Identify what the user is building (see Decision Router below).
3. Read **only** the matching `references/*.md` file(s) for detailed guidance.
4. Check the **AI pitfalls** table before generating any code.
5. Apply the Core Engineering Principles in every artifact you produce.
6. When multiple domains intersect (e.g. Nav2 + ros2_control), read both files
   but favor safety > determinism > simplicity when recommendations conflict.

**Execution log:** The Stop hook automatically appends a session summary to
`.skill-runs.log` in the workspace. This lets you see what was validated last
time and what issues were found — check it to avoid repeating past mistakes.

## Decision router

| User is doing...                                  | Read                              |
|---------------------------------------------------|-----------------------------------|
| Creating a workspace, package, or build config    | `references/workspace-build.md`   |
| Writing nodes, executors, callback groups         | `references/nodes-executors.md`   |
| Topics, services, actions, custom interfaces, QoS | `references/communication.md`     |
| Lifecycle nodes, component loading, composition   | `references/lifecycle-components.md` |
| Launch files, conditional logic, event handlers   | `references/launch-system.md`     |
| tf2, URDF, xacro, robot_state_publisher           | `references/tf2-urdf.md`         |
| ros2_control, hardware interfaces, controllers    | `references/hardware-interface.md` |
| Real-time constraints, PREEMPT_RT, memory, jitter | `references/realtime.md`         |
| Nav2, SLAM, costmaps, behavior trees              | `references/navigation.md`       |
| MoveIt 2, planning scene, grasp pipelines         | `references/manipulation.md`     |
| Camera, LiDAR, PCL, cv_bridge, depth processing   | `references/perception.md`       |
| Unit tests, integration tests, launch_testing, CI | `references/testing.md`          |
| ros2 doctor, tracing, profiling, rosbag2, CLI cheat sheet | `references/debugging.md` |
| Docker, cross-compile, fleet deployment, OTA      | `references/deployment.md`       |
| Gazebo, Isaac Sim, sim-to-real, use_sim_time      | `references/simulation.md`       |
| SROS2, DDS security, certificates, supply chain   | `references/security.md`         |
| micro-ROS, MCU/RTOS, XRCE-DDS, rclc              | `references/micro-ros.md`        |
| Multi-robot fleet, Open-RMF, DDS discovery scale  | `references/multi-robot.md`      |
| Message types, units, covariance, frame conventions | `references/message-types.md`    |
| ROS 1 migration, ros1_bridge, hybrid operation    | `references/migration-ros1.md`   |

**Cross-cutting concerns:** Security, error handling, and QoS are not isolated to
single reference files — apply them whenever the data path crosses a trust boundary,
a node owns hardware, or communication reliability matters. Use your judgment about
which cross-cutting concerns apply to the user's specific situation.

## Core engineering principles

These apply to every ROS 2 artifact you produce, regardless of domain.

### 1. Distro awareness

<!-- LAST_UPDATED: 2026-03-30 — Review this table every 6 months or when a new distro is released. -->
<!-- NEXT_REVIEW: 2026-09-30 -->
> **Staleness warning:** The table below was last verified on **2026-03-30**.
> If the current date is more than 6 months past that, re-verify EOL dates and
> feature support against https://docs.ros.org/en/rolling/Releases.html before
> relying on this table. When you update it, change both `LAST_UPDATED` and
> `NEXT_REVIEW` comments above.

Always ask which ROS 2 distribution the user targets. Key differences:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbwls99706/ros2-engineering-skills](https://github.com/dbwls99706/ros2-engineering-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

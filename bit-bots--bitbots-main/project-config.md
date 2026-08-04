---
trigger: always_on
description: This repository contains the Hamburg Bit-Bots software stack for RoboCup
---

# Agent Instructions

## Workspace Overview

This repository contains the Hamburg Bit-Bots software stack for RoboCup
humanoid soccer robots. It is a ROS 2 workspace with packages for behavior,
motion, navigation, perception, world modeling, team communication, simulation,
robot support, shared messages, and operational tooling. Packages are grouped by
capability under `src/`; reusable or externally maintained projects are imported
under `src/lib`.

The stack is implemented primarily in C++ and Python, with selected Rust
components. ROS 2 packages use `ament_cmake` or Python packaging and are built and
tested with `colcon` through Pixi tasks. Pixi and RoboStack provide the
reproducible ROS and development environment. Common foundational technologies
include:

- ROS 2 client libraries, launch, parameters, actions, services, topics, TF, and
  plugin infrastructure.
- CMake, Eigen, OpenCV, and pybind11 for native robotics and vision code.
- Python with pytest, mypy, Ruff, and ROS Python tooling.
- The Dynamic Stack Decider for behavior and state-machine-like control.
- MuJoCo for simulation and ONNX Runtime for learned models where applicable.
- pre-commit, clang-format, and cppcheck for repository-wide quality checks
  (available through Pixi).

Do not assume every package uses every technology. Inspect the affected
package's manifests and nearby code before choosing tools or patterns.

## Working Principles

- Read the affected package and its surrounding code before editing.
  Prefer established package patterns, helper APIs, and naming conventions.
- Keep changes scoped to the requested behavior.
  Do not perform unrelated refactors or reformat unrelated files.
- Preserve existing user changes in a dirty worktree.
  Never discard or overwrite changes that are unrelated to the task.
- Inspect `package.xml`, `CMakeLists.txt`, `setup.py`, and `setup.cfg` as
  applicable before changing a package's build, dependencies, or entry points.
- Update documentation, configuration examples, and tests when changing public
  behavior, parameters, interfaces, or developer workflows.
- When working on code comments or documentation, do not write down concrete values,
  as they might change later, thus making the documentation outdated.
  Instead, describe the expected behavior or refer to the relevant code sections.

## Development Environment

This ROS 2 workspace is managed by Pixi. Run development commands through the
repository's Pixi environments; do not invoke `colcon`, ROS 2 tools, or formatters
directly from the host shell.

- Use the `default` environment for normal development.
  It contains the `ros` and `format` features.
- Use the `format` environment only for formatting-only work.
- Use the `robot` environment only when robot-specific dependencies are needed.
- Prefer `pixi run -e <environment> <command>` over `pixi shell`.
  A persistent shell can become stale after environment changes.
- Use `pixi task list` to inspect available repository tasks.
- Do not manually `source install/setup.bash` (or `local_setup.bash`) inside a
  Pixi environment. Pixi's shell activation already sources the workspace
  overlay, so `ros2`, built executables, and package resolution work directly.

Common commands:

- Build the workspace with `pixi run -e default build`.
  Use the argument `--parallel-workers 2` for resource constrained environments
  (< 8 CPU cores, < 8 GB unused RAM), but prefer the default parallelism on CI
  and powerful developer machines.
- Build selected packages with
  `pixi run -e default build --packages-select <package...>`.
- Run all tests with `pixi run -e default test`.
- Test selected packages with
  `pixi run -e default test --packages-select <package...>`.
- Run formatting and linting with `pixi run -e default format`.
  Review the resulting diff because this task may modify files.
- Run one-off tools with `pixi run -e default <command>`.
- Clean all workspace build artifacts with `pixi run -e default clean`.
- Clean one package with `pixi run -e default clean <package>`.
- Use `pixi clean` only to reset Pixi's local environment data.
  This requires downloading dependencies and rebuilding afterward.

The Pixi environments provide the pinned compiler, ROS 2 installation,
dependencies, activation variables, and workspace setup used by CI. Direct host
commands may use incompatible installations or incomplete environment state.

## Dependencies

- Search the configured channels first with `pixi search <package>`.
- Determine to which section a dependency belongs to in `pixi.toml`.
- Before adding or moving a dependency, propose the suitable feature and
  environment to the user, explain why, and ask for confirmation. Do not edit
  dependency declarations or regenerate the lockfile until the user confirms.
- Prefer Conda dependencies over PyPI dependencies when a suitable package is
  available on the configured channels.
- Keep version constraints consistent with neighboring entries and explain any
  new pin or upper bound in a comment when it is not self-evident.

## ROS Packages

- Follow the package's existing Python or CMake structure rather than creating a
  new layout.
- Keep `package.xml`, build-system declarations, exports, and runtime imports in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bit-bots/bitbots_main](https://github.com/bit-bots/bitbots_main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

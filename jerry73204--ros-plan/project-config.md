---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ROS-Plan is a comprehensive ROS 2 toolchain that provides:
1. **Declarative plan format** - YAML-based node configuration with socket/link abstraction
2. **Plan compiler** - Transforms plans into executable ROS 2 launch files
3. **Runtime system** - Process management with graceful shutdown, restart policies, and dynamic parameter updates
4. **Metadata extraction** - launch2dump tool for analyzing existing launch files

The project consists of both Rust (core compiler and runtime) and Python (launch file integration) components.

## Project Status

**Current Phase**: Phase 12 (Launch-to-Plan Conversion) - In Progress
- ✅ Phase 8 Complete: Metadata Tracking (13 tests)
- ✅ 440 tests passing (330 Rust + 25 launch2dump + 9 ros2-introspect + 85 launch2plan)
- ✅ Zero warnings (clippy + compiler)
- ✅ Dependency checking with explicit error messages
- ✅ 54 features implemented across 11 phases (core system)

**Phase 12 Implementation Progress**:
- ✅ Phase 1-5: Foundation, introspection, socket inference, argument handling, plan generation
- ✅ Phase 6: Conditional branch exploration with `when` clauses
- ✅ Phase 7: Include handling & plan hierarchy (8 tests)
- ✅ Phase 8: Metadata tracking with TODO completion detection (13 tests)
- ⏳ Phase 9-12: Validation, end-to-end testing, QoS, recursive conversion (not yet started)
- See `book/src/launch2plan.md` for complete design and progress tracking

## Core Architecture

### Rust Crates (Workspace Structure)

1. **ros-plan-format** - Data structures and parsing for ROS plan files (YAML format)
   - Handles node definitions, links, sockets, parameters, QoS settings
   - Key types: Plan, Node, Link, NodeSocket, PlanSocket
   - Custom YAML tags: !pub, !sub, !pubsub, !i64, !f64, !bool, etc.

2. **ros-plan-compiler** - Core compilation logic
   - Main entry point: `Compiler::compile()`
   - Evaluation, linking, program generation
   - Lua integration for dynamic expressions
   - Link resolution and validation

3. **ros-plan-runtime** - Process management and lifecycle
   - Node spawning with ROS 2 arguments
   - Graceful shutdown (SIGTERM → SIGKILL)
   - Restart policies: Never, Always, OnFailure
   - Dynamic parameter updates with program diffing
   - Launch include tracking and reloading
   - Status reporting (table and JSON formats)
   - Event logging with circular buffer
   - Metrics collection (starts, stops, crashes, restarts)

4. **ros2plan** - CLI tool
   - Commands: `compile`, `run` (with runtime)
   - Usage: `ros2plan compile <plan-file> [args]`
   - Argument passing: `key=value` or `key:type=value`

5. **ros-utils** - ROS 2 integration utilities
   - Package and executable resolution via `ros2` CLI
   - Node command-line generation with remapping
   - ROS 2 conventions compliance

### Python Components

- **launch2dump** - Metadata extraction from ROS 2 launch files
  - Visits launch entities without spawning processes
  - Extracts nodes, parameters, remappings, containers
  - JSON/YAML output formats
  - Uses UV for dependency management (migrated from Rye)

- **ros2-introspect** - ROS 2 node interface introspection
  - Discovers publishers, subscriptions, services, clients without middleware
  - Uses rmw_introspect_cpp RMW implementation
  - Dependency checking with clear error messages
  - Requires workspace to be built and sourced

- **launch2plan** - Launch-to-Plan conversion tool (Phase 12, in progress)
  - Converts ROS 2 launch files to ROS-Plan YAML format
  - Socket inference from introspection and remappings
  - Conditional branch exploration with `when` clauses
  - Automatic link generation from topic connections
  - Argument type inference from default values
  - Include handling with cycle detection and argument forwarding
  - Metadata tracking with TODO completion detection (JSONPath addressing)
  - CLI commands: `convert` (generates plan + metadata), `status` (shows TODO progress)

### External Dependencies (Git Submodules)

- **rmw_introspect** - Located at `external/rmw_introspect/` (git submodule)
  - Separately maintained at https://github.com/jerry73204/rmw_introspect
  - Contains two packages:
    - `rmw_introspect_cpp/` - C++ RMW implementation for lightweight introspection
    - `ros2_introspect/` - Python wrapper for the RMW implementation
  - **Integration with build system**:
    - `ros2/rmw_introspect_cpp` is a symlink to `external/rmw_introspect/rmw_introspect_cpp`
    - Built automatically via `make build` using colcon
    - `ros2-introspect` (Python) added to UV workspace via `python/pyproject.toml`
  - **Dependency chain**:
    - launch2plan (Python) → ros2-introspect (Python) → rmw_introspect_cpp (ROS 2 C++)
    - launch2plan tests require workspace sourced (for rmw_introspect_cpp)
  - Required by launch2plan for node interface introspection without middleware

## Key File Formats

### Plan Files (*.yaml)
Plans define ROS 2 nodes and their connections. Example structure:
```yaml
node:
  node_name:
    pkg: package_name
    exec: executable_name
    socket:
      socket_name: !pub/!sub
link:
  link_name: !pubsub
    type: msg/type/Name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jerry73204) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

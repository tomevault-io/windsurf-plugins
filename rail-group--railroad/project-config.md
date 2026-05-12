---
trigger: always_on
description: Provides abstractions for planning execution:
---

# CLAUDE.md

`railroad` is a research repository for concurrent multi-agent probabilistic planning via a PDDL-style interface. Actions, defined via operators, contain timestamped effects. The 'state' tracks both 'active fluents' (things that are true now) and 'upcoming effects', both deterministic and probabilistic, which are effects that will become true at a later time, allowing for multi-agent concurrent action in the environment. The system combines a C++-accelerated planning core with Python bindings, optionally integrated with AI2-THOR/ProcTHOR simulators for environment generation.

## Build System

This project uses `uv` as the package manager and build tool. The build system automatically handles C++ compilation when needed. Build is automatic via `uv run`, which detects changes to code (including C++) and rebuilds as necessary. Do not run `uv sync` unless explicitly needed.

- **Type checking**: `uv run ty check`. Type checking is fast and effective and should be run before tests, examples, or scripts.
- **Unit tests**: `uv run pytest`. Tests can be filtered via `uv run pytest -vk <filter>`. We have strong test coverage, so tests are a good way to validate the code is working. For quick passes use `uv run pytest -m 'not slow'`, which takes around 10 seconds instead of 1 minute.
- **Run an example planning problem**: `uv run railroad example <name>` (e.g., `clear-table`, `multi-object-search`, `heterogeneous-robots`)

## Architecture

The repository is organized as a monorepo with several interdependent packages:

#### Core Planning (`packages/railroad/`)

- **C++ Core** (`include/`, `src/railroad/_bindings.cpp`):
  - A* search, MCTS planning
  - State management and action grounding
  - FF-like heuristic for forward planning
- **Python Layer** (`src/railroad/`):
  - `core.py`: Main classes (`Fluent`, `State`, `Action`, `Effect`, `Operator`, `Goal`) - re-exports C++ types and adds Python utilities
  - `operators/`: Helper functions to construct common operators (move, search, pick, place, wait)
  - `planner.py`: `MCTSPlanner` wrapper with automatic negative precondition handling
- **Testing**: Tests in `tests/` including unit tests and integration tests

#### Environment Module (`packages/railroad/src/railroad/environment/`)

Provides abstractions for planning execution:

- **`environment.py`**: `Environment` abstract base class
  - Active skill tracking and time management
  - State assembly (fluents + upcoming effects)
  - Action instantiation from operators
  - The `act()` loop that executes until a robot is free
  - `ActiveSkill` protocol for skill execution
  - Subclass hooks: `define_operators()`, `set_robot_pose()`, `_should_interrupt_skills()`, `_cap_next_advance_time()`, `_after_skills_advanced()`

- **`symbolic.py`**: `SymbolicEnvironment` and skill implementations
  - `SymbolicEnvironment`: Concrete environment for symbolic execution
  - `SymbolicSkill`: Standard skill implementation (non-interruptible)
  - `LocationRegistry`: Coordinates robot locations during interruptible moves

- **`skill/`**: Skill protocols and navigation skill implementations
  - `protocols.py`: `MotionSkill` protocol, `SupportsMovePathEnvironment` contract
  - `navigation.py`: `NavigationMoveSkill` (path-following with occupancy-grid pathing), `InterruptibleNavigationMoveSkill` (interruptible variant)

- **`types.py`**: Shared types (`Pose`, `PoseLike` protocol)

- **`procthor/`**: ProcTHOR simulator interface (optional dependency)
  - `ThorInterface`: Main underlying interface to AI2-THOR/ProcTHOR scenes
  - `ProcTHORScene`: User-facing data provider for ProcTHOR, wrapping ThorInterface
  - `ProcTHOREnvironment`: Subclass of `SymbolicEnvironment` + `OccupancyGridPathingMixin`; subclasses must override `define_operators()`
  - `SceneGraph`: Scene graph representation

#### Navigation Module (`packages/railroad/src/railroad/navigation/`)

Reusable grid navigation primitives, independent of any specific environment:

- `pathing.py`: Theta\* any-angle pathfinding, Dijkstra cost grids, path interpolation, grid inflation
- `occupancy_grid_mixin.py`: `OccupancyGridPathingMixin` — mixin providing `estimate_move_time()` and `compute_move_path()` from an occupancy grid
- `plotting.py`: Occupancy grid visualization helpers
- `constants.py`: Grid cell values (`COLLISION_VAL`, `FREE_VAL`, `UNOBSERVED_VAL`, `OBSTACLE_THRESHOLD`)

#### Benchmarking (`packages/railroad/src/railroad/bench/`)
- `registry.py`: Benchmark registration via `@benchmark` decorator
- `runner.py`: Parallel benchmark execution with MLflow tracking
- `dashboard/`: Interactive Plotly Dash visualization
- `benchmarks/`: Benchmark definitions (multi-object search, movie night, etc.)

#### Experimental (`packages/railroad/src/railroad/experimental/`)
- **`unknown_search/`**: Frontier-based unknown-space exploration and object search
  - `UnknownSpaceEnvironment`: occupancy-grid environment with laser sensing, frontier detection, and navigation skills
  - Specialized operators for navigable moves, site search, and frontier search

#### External Packages
- **`packages/environments/`**: Additional environment implementations (PyRoboSim)

### Key Concepts

#### Creating a New Planning Problem

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RAIL-group/railroad](https://github.com/RAIL-group/railroad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

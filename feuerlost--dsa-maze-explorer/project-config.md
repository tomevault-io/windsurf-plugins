---
trigger: always_on
description: This is a Data Structures and Algorithms course design project.
---

# AGENTS.md

## Project Overview

This is a Data Structures and Algorithms course design project.

Topic: Maze Solving / 走迷宫

The project should implement the following features:

1. Rectangular maze representation, generation, and path search.
2. DFS for finding any valid path.
3. BFS for finding the shortest path in an unweighted maze.
4. Pygame GUI for displaying the maze, the search process, and the final path.
5. Mondrian artwork-to-maze conversion:
   - Do not hard-code a fixed Mondrian layout.
   - Implement image processing scripts to detect black thick lines and colored rectangular regions from a Mondrian reference image.
   - Generate `configs/mondrian_layout.json`.
6. Circular non-rectangular maze generation and path search.
7. Pytest tests.
8. README, report draft, and demo video script.

The main design idea is to model every maze as a graph. Nodes represent walkable cells or regions, and edges represent valid movement between adjacent positions. Concrete maze types are responsible for generating nodes and neighbor relationships, while DFS, BFS, and constrained BFS should be implemented in a shared algorithm layer.

## Development Environment

The project is developed in WSL.

Use conda for the Python environment and pip for dependencies.

```bash
conda create -n dsa_maze python=3.10 -y
conda activate dsa_maze
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Main dependencies:

```text
pygame
pytest
numpy
opencv-python
pillow
```

The recommended `requirements.txt` is:

```text
pygame>=2.5.0
pytest>=8.0.0
numpy>=1.24.0
opencv-python>=4.8.0
pillow>=10.0.0
```

## Architecture Requirements

Use a unified `MazeBase` abstraction.

Search algorithms must be decoupled from concrete maze types.

Expected project structure:

```text
dsa-maze-explorer/
├── AGENTS.md
├── main.py
├── requirements.txt
├── README.md
├── logs/
│   └── codex_log.md
├── assets/
│   └── mondrian/
│       ├── README.md
│       └── reference.png
├── configs/
│   ├── rect_default.json
│   ├── circular_default.json
│   └── mondrian_layout.json
├── src/
│   ├── core/
│   │   ├── __init__.py
│   │   ├── maze_base.py
│   │   ├── search.py
│   │   └── path_result.py
│   ├── mazes/
│   │   ├── __init__.py
│   │   ├── rectangular.py
│   │   ├── mondrian.py
│   │   └── circular.py
│   ├── gui/
│   │   ├── __init__.py
│   │   ├── app.py
│   │   ├── draw_rectangular.py
│   │   ├── draw_mondrian.py
│   │   └── draw_circular.py
│   └── utils/
│       ├── __init__.py
│       └── config.py
├── tools/
│   ├── extract_mondrian_layout.py
│   ├── validate_mondrian_layout.py
│   └── preview_mondrian_layout.py
├── tests/
│   ├── test_search.py
│   ├── test_rectangular.py
│   ├── test_mondrian.py
│   └── test_circular.py
└── report/
    ├── report.md
    ├── video_script.md
    └── images/
```

Each maze type should implement the following interface through `MazeBase`:

```python
nodes()
neighbors(node)
start
goal
```

DFS, BFS, and Mondrian room-constrained BFS should live in `src/core/search.py`.

## Core Modules

### `src/core/maze_base.py`

Define the abstract base class for all mazes.

Expected responsibilities:

- Provide `start` and `goal`.
- Provide `nodes()`.
- Provide `neighbors(node)`.
- Keep the interface independent of specific maze shapes.

### `src/core/path_result.py`

Define a `PathResult` data class.

Expected fields:

```python
path: list
visited_order: list
found: bool
algorithm: str
extra: dict
```

### `src/core/search.py`

Implement shared graph search algorithms:

- `dfs_find_path(maze)` for finding any valid path.
- `bfs_shortest_path(maze)` for finding the shortest path in an unweighted maze.
- `bfs_with_room_constraint(maze, min_rooms_required=3)` for Mondrian maze search.

Search functions should return `PathResult`.

## Maze Modules

### Rectangular Maze

File: `src/mazes/rectangular.py`

Requirements:

- Use a `rows x cols` grid.
- Node format: `(row, col)`.
- Each cell has four walls: up, right, down, and left.
- Use randomized DFS/backtracking to generate a perfect maze.
- The entrance should be on the left boundary.
- The exit should be on the opposite right boundary.
- `neighbors(node)` should only return adjacent cells with no wall between them.
- Support a random seed for reproducible results.

### Mondrian Maze

File: `src/mazes/mondrian.py`

Requirements:

- Read layout from `configs/mondrian_layout.json`.
- Treat black thick lines in the original artwork as walls.
- Treat colored rectangular regions as rooms.
- Use doors to connect neighboring rooms.
- Discretize each room into small grid nodes.
- Recommended node format: `(room_id, local_row, local_col)`.
- Implement `room_of(node)`.
- Implement path search with the requirement that the final path passes through at least three rooms.

### Circular Maze

File: `src/mazes/circular.py`

Requirements:

- Use a polar-coordinate grid.
- Node format: `(ring, sector)`.
- Use a configurable number of rings and sectors.
- Each node may connect to:
  - left sector in the same ring,
  - right sector in the same ring,
  - inner ring,
  - outer ring.
- Use randomized DFS/backtracking to generate the maze.
- Put the entrance on the outer boundary.
- Put the exit inside the circular maze.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feuerlost/dsa-maze-explorer](https://github.com/feuerlost/dsa-maze-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->

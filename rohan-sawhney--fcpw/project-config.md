---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FCPW (Fastest Closest Points in the West) is a header-only C++ library for high-performance geometric queries on 2D line segment meshes and 3D triangle meshes. It provides both CPU vectorization (via Enoki SIMD) and GPU acceleration (via Slang shading language).

**Key Query Types:**
- Closest point queries
- Ray intersections
- Silhouette queries (visibility-based)
- Containment testing
- Line-of-sight checks
- CSG operations (union, intersection, difference)

## Build Commands

### Initial Setup
```bash
# Clone with submodules
git submodule update --init --recursive

# Clone optional dependencies for demos
git clone --recurse-submodules https://github.com/nmwsharp/polyscope.git deps/polyscope
git clone --recurse-submodules https://github.com/wjakob/nanobind.git deps/nanobind
```

### C++ Build
```bash
mkdir build && cd build

# Basic build with demos and tests
cmake -DFCPW_BUILD_DEMO=ON -DFCPW_BUILD_TESTS=ON ..

# Build with GPU support enabled
cmake -DFCPW_BUILD_DEMO=ON -DFCPW_BUILD_TESTS=ON -DFCPW_ENABLE_GPU_SUPPORT=ON ..

# Build without Enoki (falls back to Eigen, no vectorization)
cmake -DFCPW_USE_ENOKI=OFF ..

# Build with 8-wide branching (default is 4-wide)
cmake -DFCPW_USE_EIGHT_WIDE_BRANCHING=ON ..

make -j8
```

**CMake Options:**
- `FCPW_USE_ENOKI` (ON/OFF) - Enable CPU SIMD vectorization
- `FCPW_ENABLE_GPU_SUPPORT` (ON/OFF) - Enable GPU acceleration
- `FCPW_BUILD_DEMO` (ON/OFF) - Build interactive demo
- `FCPW_BUILD_TESTS` (ON/OFF) - Build test suite
- `FCPW_BUILD_BINDINGS` (ON/OFF) - Build Python bindings
- `FCPW_USE_EIGHT_WIDE_BRANCHING` (ON/OFF) - Use 8-wide MBVH instead of 4-wide

### Python Build
```bash
# Install from PyPI
pip install fcpw

# Build locally (without GPU support by default)
pip install .

# Build with GPU support
pip install . --config-settings=cmake.define.FCPW_ENABLE_GPU_SUPPORT=ON
```

## Running Tests

### C++ Tests (from build/ directory)

**3D Triangle Mesh Tests - Correctness:**
```bash
./tests/aggregate_tests --dim=3 --tFile ../tests/input/bunny.obj --nQueries=1024 --checkCorrectness --plotInteriorPoints --computeSilhouettes --vizScene
./tests/aggregate_tests --dim=3 --tFile ../tests/input/armadillo.obj --nQueries=1024 --checkCorrectness
```

**3D Triangle Mesh Tests - Performance:**
```bash
./tests/aggregate_tests --dim=3 --tFile ../tests/input/bunny.obj --nQueries=1048576 --checkPerformance
```

**2D Line Segment Tests:**
```bash
./tests/aggregate_tests --dim=2 --lFile ../tests/input/plus-shape.obj --nQueries=1024 --checkCorrectness --vizScene
./tests/aggregate_tests --dim=2 --lFile ../tests/input/spiral.obj --nQueries=1048576 --checkPerformance
```

**CSG Tests:**
```bash
./tests/csg_tests --dim=3 --tFile ../tests/input/armadillo.obj --tFile ../tests/input/bunny.obj --tFile ../tests/input/kitten.obj --csgFile ../tests/input/csg.txt --instanceFile ../tests/input/instances3d.txt
```

**GPU Tests:**
```bash
./tests/gpu_tests --dim=3 --tFile ../tests/input/bunny.obj --nQueries=1048576 --plotInteriorPoints --vizScene
./tests/gpu_tests --dim=2 --lFile ../tests/input/spiral.obj --nQueries=1048576 --vizScene
```

### Python Tests (from tests/ directory)
```bash
# Basic CPU tests
python3 fcpw_tests.py --file_path=input/bunny.obj --dim=3 --n_queries=1024 --compare_with_cpu_baseline --visualize_scene

# GPU tests
python3 fcpw_tests.py --file_path=input/bunny.obj --dim=3 --n_queries=1048576 --run_gpu_queries --compare_with_warp --visualize_scene
```

### Running Demos
```bash
# From build/ directory
./demos/demo [--useGpu]

# Python demo (from demos/ directory, requires polyscope)
python -m pip install polyscope
python demo.py [--use_gpu]
```

## Python API Performance

### Efficient Interaction Data Extraction

When working with large numbers of intersection or closest point queries, extracting data from `interaction_list` objects can become a bottleneck if done with Python loops. FCPW provides **bulk extraction methods** that are 3-10x faster than manual iteration.

**❌ Slow - Python Loop (Avoid This):**
```python
# Extracting data element-by-element is slow
distances = np.empty(len(interactions))
normals = np.empty((3, len(interactions)))
for i, interaction in enumerate(interactions):
    distances[i] = interaction.d
    normals[:, i] = [interaction.n[0], interaction.n[1], interaction.n[2]]
```

**✅ Fast - Bulk Extraction (Use This):**
```python
# Extract all data at once in C++ (3-10x faster!)
distances = interactions.get_distances()          # shape: (n,)
positions = interactions.get_positions()          # shape: (n, 3) or (n, 2)
normals = interactions.get_normals()              # shape: (n, 3) or (n, 2)
uvs = interactions.get_uvs()                      # shape: (n, 2) or (n, 1)
indices = interactions.get_primitive_indices()    # shape: (n,)
```

**Available Methods:**
- `interaction_2D_list`: Returns arrays with shape (n, 2) for positions/normals, (n, 1) for uvs
- `interaction_3D_list`: Returns arrays with shape (n, 3) for positions/normals, (n, 2) for uvs
- `gpu_interaction_list`: Returns arrays with shape (n, 3) for positions/normals, (n, 2) for uvs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rohan-sawhney/fcpw](https://github.com/rohan-sawhney/fcpw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

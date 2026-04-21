---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Quick Start**: See [AGENTS.md](AGENTS.md) for the AI agent entry point and session protocol.

## Project Overview

CCV (Chenguang Computer Vision) is a C++ computer vision library with zero third-party dependencies in core logic. It provides foundational mathematics, kinematics, dynamics, estimation, and core CV algorithms. The repository also contains wrappers and demos in Python, Java, C#, and MATLAB.

**Key architectural principles:**
- Core algorithm code has **no dependencies** (no OpenCV, Eigen, or PCL in core logic)
- **C++17 or higher** required (enforced in CMakeLists.txt)
- Uses `FLOAT` macro to toggle between single/double precision (defaults to `float`)
- Uses `eps` constant for safe floating-point comparisons near zero
- All code is in the `cg` namespace
- Mathematical formulas are documented in Doxygen comments with LaTeX

**Version Requirements:**
- **C++**: C++17 or higher
- **Python**: 3.10 or higher (for Python bindings)

## Build Commands

### Quick Start (Recommended)
```bash
# Initialize environment
./scripts/init.sh

# Build with sensible defaults (Release, tests ON, OpenCV ON)
./scripts/build.sh

# Run all tests
./scripts/test.sh

# Full validation before commit
./scripts/check.sh
```

### CMake Build (Standalone)
```bash
# Configure with tests enabled
cmake -S ./ -B build -DBUILD_TEST=ON

# Build with parallel jobs
cmake --build build --target install/strip --parallel $(($(nproc) / 4))

# Optional flags:
#   -DBUILD_TEST=ON      # Build unit tests (requires GTest)
#   -DBUILD_DOCS=ON      # Build Doxygen documentation
#   -DWITH_OPENCV=ON     # Enable OpenCV integration
#   -DWITH_PCL=ON        # Enable PCL integration
#   -DWITH_GL=ON         # Enable OpenGL/Pangolin
```

### CMake Presets (Convenient Configurations)
```bash
# List available presets
cmake --list-presets

# Use a preset
cmake --preset=release    # Release build with tests
cmake --preset=debug      # Debug build
cmake --preset=minimal    # No external dependencies
cmake --preset=full       # All features including docs

# Build using preset
cmake --build --preset=release
ctest --preset=release
```

### ROS Build (catkin_tools)
```bash
catkin build -j$(nproc) -DWITH_ROS=ON -DWITH_PCL=ON ccv
```

### Python Package
**Requirement: Python 3.10 or higher**

```bash
cd python
pip install .

# With optional dependencies
pip install "libccv[torch]"  # PyTorch support
pip install "libccv[yolo]"   # YOLO/Ultralytics
pip install "libccv[mp]"     # MediaPipe
```

## Test Commands

### Run All Tests
```bash
cd build
ctest --output-on-failure
```

### Run Individual Test Executables
```bash
# Maths tests
./unit_test/test_maths

# CV tests
./unit_test/test_cv

# Kinematics tests
./unit_test/test_kinematics
```

## Linting / Formatting

### Automated Scripts
```bash
# Format all code (C++ and Python)
./scripts/format.sh

# Full validation check (format, build, test)
./scripts/check.sh
```

### Manual Formatting

#### C++ Code Formatting
```bash
# Format all C++ files with clang-format (Google style, 120 column limit)
find . -name "*.cpp" -o -name "*.cc" -o -name "*.h" -o -name "*.hpp" | xargs clang-format -i
```

#### Python Code Formatting
```bash
# Ruff is configured in .pre-commit-config.yaml
cd python
ruff check .
ruff format .
```

## Architecture

### Module Structure

**Three core libraries are built:**
1. `libccv_maths.so` - Matrix, vector, SVD, random numbers (`src/maths/`)
2. `libccv_kd.so` - Kinematics & dynamics: rotation matrices, quaternions, Euler angles (`src/kd/`)
3. `libccv.so` - Computer vision: image processing, features, calibration (`src/cv/`)

**Library dependency chain:** `ccv` → `ccv_kd` → `ccv_maths`

### Directory Layout

```
include/ccv/          # Public headers
  cv/                 # Image, features, camera types
  maths/              # Matrix, vector, math types
  kd/                 # Rotation, quaternion, transform types

src/                  # Implementation
  maths/              # Matrix, vector, SVD implementations
  cv/                 # Image processing, features2d
  kd/                 # Rotation matrices, quaternions, converters
  est/                # Estimation: EKF, filters, bundle adjustment

unit_test/            # GTest-based unit tests
  test_math.cpp
  test_cv.cpp
  test_kinematics.cpp

apps/                 # Applications
  camkit/             # Camera utilities (V4L2, MIPI, RealSense, ROS)
  stereo_match/       # Stereo matching demos
  stereo_reconstruct/ # 3D reconstruction
  visual_tracking/    # Visual tracking demos

libs/                 # Third-party integration demos
  opencv/             # OpenCV integration examples
  pcl/                # Point Cloud Library demos
  pangolin/           # SLAM visualization
  opengl/             # OpenGL demos
  fftw/               # FFT examples

python/               # Python package (libccv on PyPI)
  libccv/             # Python modules

hpc/                  # High-performance computing (currently empty)
```

### Key Type Definitions

- `hpc::TScalarF` - Configurable float type (float or double via FLOAT macro)
- `cg::Point2i`, `cg::Point2f`, `cg::Point2d` - 2D points

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cggos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

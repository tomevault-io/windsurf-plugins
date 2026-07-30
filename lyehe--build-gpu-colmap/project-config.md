---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Point Cloud Tools** - Self-contained build environment for COLMAP and related computer vision tools with CUDA support. This project uses vcpkg for dependency management and CMake's ExternalProject module to orchestrate builds of multiple interdependent computer vision libraries.

**Note:** GLOMAP has been merged into COLMAP. Use `colmap global_mapper` for global Structure-from-Motion.

### Key Components
- **COLMAP 4.1.0** - Structure-from-Motion and Multi-View Stereo (general use)
- **COLMAP 4.1.0 for pycolmap** (colmap-for-pycolmap) - Configured specifically for Python wheel building
- **Ceres Solver** - Nonlinear optimization library (base dependency)

## Common Build Commands

### Initial Setup
```powershell
# Windows - Verify build environment
.\scripts_windows\verify_build_environment.ps1
```

```bash
# Linux
./scripts_linux/verify_build_environment.sh
```

**Note:** All build scripts automatically initialize required submodules and bootstrap vcpkg if not already done. You can start building immediately after cloning the repository.

### Building

```powershell
# Windows - Build COLMAP
# Automatically initializes submodules and bootstraps vcpkg
.\scripts_windows\build.ps1 -Configuration Release

# Build only COLMAP (latest)
.\scripts_windows\build_colmap.ps1

# Build without CUDA
.\scripts_windows\build.ps1 -NoCuda

# Clean rebuild
.\scripts_windows\build.ps1 -Clean
```

```bash
# Linux
./scripts_linux/build.sh --config Release
./scripts_linux/build_colmap.sh
./scripts_linux/build.sh --no-cuda
./scripts_linux/build.sh --clean
```

### Python Wheels

**Option 1: Use regular COLMAP (default)**
```powershell
# Windows - Build wheels for ALL installed Python 3.9+ versions
.\scripts_windows\build_colmap.ps1
.\scripts_windows\build_pycolmap_wheels.ps1
```

```bash
# Linux - All versions (recommended)
./scripts_linux/build_colmap.sh
./scripts_linux/build_pycolmap_wheels.sh

# Linux - Single version (alternative)
./scripts_linux/build_pycolmap_wheel.sh
```

**Option 2: Use COLMAP-for-pycolmap (optimized configuration)**
```bash
# Build COLMAP with pycolmap-specific configuration
mkdir build && cd build
cmake .. \
  -DCMAKE_TOOLCHAIN_FILE=../third_party/vcpkg/scripts/buildsystems/vcpkg.cmake \
  -DBUILD_COLMAP=OFF \
  -DBUILD_COLMAP_FOR_PYCOLMAP=ON \
  -DCUDA_ENABLED=ON
cmake --build . --config Release

# Then use the install from build/install/colmap-for-pycolmap/
```

## Architecture & Build System

### Multiple COLMAP Version Strategy
This project can build **two separate COLMAP versions**:

1. **COLMAP 4.1.0** (`third_party/colmap`) → `build/install/colmap/`
   - For general use
   - COLMAP 4.1.0 release
   - Full features enabled
   - Includes global SfM (previously GLOMAP): `colmap global_mapper`

2. **COLMAP 4.1.0 for pycolmap** (`third_party/colmap-for-pycolmap`) → `build/install/colmap-for-pycolmap/`
   - For building Python wheels with specific configurations
   - COLMAP 4.1.0 release
   - Built with GUI/tests disabled, optimized for Python bindings
   - Optional build (set `-DBUILD_COLMAP_FOR_PYCOLMAP=ON`)

**Why?** Different use cases require different COLMAP configurations for optimal compatibility and performance.

### Build Dependency Chain

The CMakeLists.txt uses `ExternalProject_Add()` to enforce strict build order:

```
Ceres Solver (base dependency)
    ├── COLMAP 4.1.0 ───────────────────→ build/install/colmap/
    └── COLMAP 4.1.0 for pycolmap (optional) → build/install/colmap-for-pycolmap/
```

**Critical Details:**
- Ceres is built first with `ExternalProject_Add()` and installed to `build/install/ceres/`
- Subsequent projects use `-DCMAKE_PREFIX_PATH=${BASE_INSTALL_DIR}/ceres` to find Ceres
- All projects share the same `vcpkg_installed` directory for dependencies

### vcpkg Integration

**Local vcpkg** (not system vcpkg):
- Submodule at `third_party/vcpkg/`
- Bootstrapped via `scripts_windows/bootstrap.ps1` or `scripts_linux/bootstrap.sh`
- Toolchain file: `third_party/vcpkg/scripts/buildsystems/vcpkg.cmake`
- Manifest mode: Dependencies defined in `vcpkg.json`
- Overlay ports in `overlay-ports/` for custom patches (e.g., SuiteSparse CUDA support)

**vcpkg Features:**
- `cuda` feature enabled when `CUDA_ENABLED=ON` (default)
- Manifest features set BEFORE `project()` in CMakeLists.txt (line 14-17)

### CUDA Configuration

**CUDA Detection Flow** (CMakeLists.txt lines 34-240):
1. Detects CUDA Toolkit via `find_package(CUDAToolkit)`
2. Sets architectures: `75;80;86;89;90` (RTX 20/30/40 series, H100)
3. Searches for cuDSS (optional sparse solver library):
   - Windows: `C:\Program Files\NVIDIA cuDSS\v*/`
   - Linux: `/usr/local/cuda/`, `/opt/nvidia/cudss/`, `$CUDSS_ROOT`
4. Exports cuDSS paths to subprojects via `CMAKE_PREFIX_PATH` and environment variables

**cuDSS Detection:**
- Version-specific lib directory: `lib/12/cudss.lib` for CUDA 12.x
- Auto-detects CUDA major version and searches for matching cuDSS libs
- Falls back gracefully if not found (2-5x performance loss for sparse solvers)

### Build Output Structure

```
build/
├── install/                    # All installation outputs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lyehe/build_gpu_colmap](https://github.com/lyehe/build_gpu_colmap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

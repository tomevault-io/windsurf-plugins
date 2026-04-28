---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ReShade addon that extracts training data from games for deep neural networks (DNN), including RGB images, depth maps, camera matrices, and semantic segmentation. It supports creating datasets for NeRF reconstruction, SLAM, monocular depth estimation, and semantic segmentation.

The addon hooks into game rendering pipelines via ReShade 5.8.0+ to capture:
- RGB frames
- Depth buffers (calibrated physical distance when supported)
- Camera extrinsic matrices (cam2world) and FOV
- Semantic segmentation data (DirectX 10/11 only)

## 重要规则

**修改完成后自动编译部署测试。** 代码修改完成后，自动执行以下流程：
1. 调用 `msbuild gcv_reshade.sln /p:Configuration=Release /p:Platform=x64` 编译
2. 编译成功后，将 `build/x64/Release/cv_captures.addon` 复制到当前游戏的可执行文件目录（从 `doc/game/<GameName>.md` 顶部获取路径）
3. 启动游戏进行测试（通过 Steam 或直接运行 exe）
如果编译失败，立即分析错误并修复，不要等用户干预。

**PR 默认提交到当前 fork（origin）。** 创建 PR 时，除非用户明确指定提交到 upstream，否则一律使用 `--repo` 指向当前 fork（即 `origin` 对应的仓库），不要提交到 upstream。

**每次更改主框架时同步更新 changelog。** 修改框架文件后，按 `/update-changelog` skill 的规范更新 `doc/CHANGELOG.md`。

## Build Commands

### Build with Visual Studio
```bash
# Open solution in Visual Studio 2022
start gcv_reshade.sln

# Or build from command line with MSBuild
msbuild gcv_reshade.sln /p:Configuration=Release /p:Platform=x64
```

The build outputs `cv_captures.addon` to `build/x64/Debug/` or `build/x64/Release/`.

### Dependencies via vcpkg

Required vcpkg packages (target `x64-windows`):
- `eigen3` - Matrix operations
- `nlohmann-json` - JSON serialization
- `xxhash` - Fast hashing

ReShade 5.8.0 source must be placed at `..\reshade-5.8.0` (one level up from this repo).

### Installation to Game

1. Install ReShade 6.4.0+ to your game directory
2. Copy the compiled `cv_captures.addon` to the game executable directory (where `ReShade.log` is)
3. For games requiring camera matrix extraction, install the appropriate mod script from `mod_scripts/` (see game-specific instructions in script comments)
4. Copy `reshade_shaders/*.fx` to the ReShade shader search path if needed

## Architecture

### High-Level Data Flow

1. **Game Rendering** → ReShade hooks intercept draw calls and frame buffers
2. **ReShade Addon (gcv_reshade)** → Captures RGB, depth, and camera data each frame
3. **Game Interface (gcv_games)** → Game-specific logic extracts camera matrix and converts depth values
4. **Writer Threads** → Serialize data to disk asynchronously (JSON + NPY + FPZIP compressed depth)
5. **Post-processing (python_threedee)** → Convert captures to NeRF transforms.json or point clouds

### Core Components

**`gcv_reshade/` - ReShade Addon Core**
- `main.cpp`: Main addon entry point with ReShade API hooks. Handles F11 (single capture) and F9/F10 (video recording start/stop) hotkeys.
- `copy_texture_into_packedbuf.cpp/.h`: Texture copying utilities for RGB and depth buffers. Supports shader-based depth capture for DX12/Vulkan via `DepthCapture.fx`.
- `grabbers.cpp/.h`: Frame buffer grabbing logic
- `recorder.cpp/.h`: Video recording state machine
- `image_writer_thread_pool.cpp/.h`: Async disk I/O for captured frames

**`gcv_games/` - Game-Specific Implementations**
- Each game has its own class (e.g., `Cyberpunk2077.cpp`, `ResidentEvils.cpp`, `HorizonZeroDawn.cpp`)
- Game classes must implement:
  - `gamename_verbose()`: Display name
  - `camera_dll_name()` / `camera_dll_mem_start()`: Where to find camera data in memory
  - `convert_to_physical_distance_depth_u64()`: Convert raw depth buffer values to meters
  - Camera matrix extraction logic (often via scripted buffer or memory scanning)
- `game_interface_factory.cpp`: Factory pattern for auto-registering game implementations

**`gcv_utils/` - Shared Utilities**
- `camera_data_struct.cpp/.h`: `CamMatrixData` struct for extrinsic matrices and FOV, serialization to JSON
- `memread.cpp/.h`: Memory reading helpers for extracting data from game process memory
- `depth_utils.h`: Depth conversion utilities
- `scripted_cam_buf_templates.h`: Templates for extracting camera data from script-injected memory buffers

**`mod_scripts/` - Game Mod Scripts** (not part of C++ build)
- Lua/CET scripts that run inside games to export camera matrices to shared memory
- Examples: `cyberpunk2077_cyberenginetweaks_mod_init.lua`, `residentevil_read_camera_matrix_transfcoords.lua`

**`python_threedee/` - Post-Processing Scripts** (not part of C++ build)
- `convert_game_snapshot_jsons_to_nerf_transformsjson.py`: Aggregate per-frame JSONs into NeRF-ready transforms.json
- `load_point_cloud.py`: Visualize captures as 3D point clouds in Open3D, save as PLY/PCD
- `_jake/run_pipeline.py`: **主重建脚本**，从采集数据重建点云的完整流水线。F9 连续录制的丢帧处理（RGB/深度/相机矩阵帧号对齐）需要在此脚本中正确剔除

**`reshade_shaders/` - Custom ReShade Effects**
- `DepthCapture.fx`: Shader for capturing depth on DX12/Vulkan where direct buffer access isn't available
- `segmentation_visualization.fx`: Real-time visualization of semantic segmentation
- `displaycamcoords.fx`: Display camera coordinates overlay

### Depth Capture Strategies

The addon supports two depth capture methods:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zikun-dai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->

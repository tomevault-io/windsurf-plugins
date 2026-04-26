---
trigger: always_on
description: This is a GPU-accelerated path tracing renderer built with DirectX 12 Raytracing (DXR) for an Advanced Computer Graphics course project. The renderer implements physically-based rendering with support for basic path tracing, diffuse/specular/transmissive materials, texture mapping, HDR environment lighting, anti-aliasing, and real-time denoising using Intel OIDN.
---

# ACG Project - AI Coding Assistant Instructions

## Project Overview
This is a GPU-accelerated path tracing renderer built with DirectX 12 Raytracing (DXR) for an Advanced Computer Graphics course project. The renderer implements physically-based rendering with support for basic path tracing, diffuse/specular/transmissive materials, texture mapping, HDR environment lighting, anti-aliasing, and real-time denoising using Intel OIDN.

### Current Implementation Status
**Completed Features:**
- Path tracing with diffuse/specular/transmissive materials
- Color texture mapping
- HDR environment lighting (skybox)
- Anti-aliasing
- Hardware-accelerated BVH (via DXR)

**In Development:**
- Scene creation and optimization
- Advanced material layers (clearcoat, sheen, subsurface)
- Normal/height/attribute maps
- Importance sampling with Russian Roulette
- Point and area lights
- Simulation-based content

**Planned (Not Yet Started):**
- Principled BSDF
- Multi-layer materials
- Adaptive mipmap
- Volumetric rendering
- Motion blur, depth of field
- Advanced visual effects

## Architecture

### Core Components
- **Renderer (`Renderer.h/cpp`)**: DirectX 12 DXR pipeline managing acceleration structures, shader binding tables, and raytracing dispatch
- **Scene (`Scene.h/cpp`)**: Container for meshes, materials, lights with async loading via Python loaders
- **Material System (`Material.h`, `MaterialLayers.h`)**: 64-byte GPU-aligned PBR materials with infrastructure for layered extensions (implementation in progress)
- **Virtual Texture System (`VirtualTextureSystem.h/cpp`)**: Tile-based texture streaming infrastructure (256×256 tiles, not yet fully integrated)
- **Denoiser (`Denoiser.h/cpp`)**: Integration with Intel Open Image Denoise for progressive rendering cleanup

### Data Flow
1. **Scene Conversion** (Separate Step): GUI conversion tool calls Python loaders → binary `.acg` format
2. **Scene Loading**: Load pre-converted `.acg` files → C++ Scene object (no automatic conversion)
3. **GPU Upload**: Scene data converted to GPU structures → uploaded to default heaps via staging buffers
4. **Acceleration Structure**: Bottom-level AS per mesh, top-level AS for scene (built using `CreateAccelerationStructures()`)
5. **Raytracing**: HLSL shaders (`shaders/Raytracing.hlsl`) trace rays using acceleration structures
6. **Output**: Accumulation buffer → optional denoising → swapchain for display or PPM file export

### CPU-GPU Data Synchronization
- **GPUMaterial** (96 bytes, `Renderer.h`): Simplified structure for shader constant buffers
- **Material** struct (64 bytes, `Structures.hlsli`): GPU-side HLSL structure matching `MaterialData` in `Material.h`
- **Vertex** (48 bytes): Position, normal, texcoord, tangent (16-byte aligned)
- Material layer flags must match between `MaterialLayers.h` and `Structures.hlsli` defines

## Build System

### Dependencies (vcpkg)
```bash
cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE="$env:VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake"
cmake --build build --config Release
```
Required packages in `vcpkg.json`: glm, imgui (with dx12-binding), stb, d3dx12, openexr, tinyexr, miniz

### Python Scene Loader Setup
```bash
# From project root - creates .venv with Python 3.11 (required for bpy)
.\setup_loader.bat
```
Python dependencies in `requirements.txt`:
- **PyWavefront** (>=1.3.3): OBJ/MTL file parsing
- **bpy** (>=5.0.0): Blender Python API for .blend files
- **numpy** (>=1.21.0): Numerical operations
- **requests** (>=2.31.0): Test scene downloading

**Scene Conversion Workflow:**
- Conversion and loading are **completely separated** - no automatic conversion during loading
- GUI has two independent sections:
  1. **Scene Conversion**: Convert OBJ/FBX/GLTF/Blend → ACG binary (calls `bin/loader/main.py`)
  2. **Scene Loading**: Load pre-converted ACG files only (no conversion)
- Python loader uses executable directory path: `<exe_dir>/loader/main.py` (not working directory)
- Python output redirected to GUI log (ERROR level only, UTF-8 encoded)
- Conversion results displayed via ImGui modal popup

### Build Artifacts
- Executable: `bin/ACG_Project.exe`
- Shaders: Auto-copied from `shaders/` → `bin/shaders/`
- DLLs: OIDN, DXC compiler (from Windows SDK), PIX (Debug only)

## Development Conventions

### Shader Development
- **Always modify `.hlsl`/`.hlsli` files** in `shaders/`, not in `bin/shaders/` (auto-copied by CMake)
- Root signature is defined programmatically in `Renderer::CreateRaytracingRootSignature()`, NOT in shader code
- Shader compilation uses DXC at runtime via `D3DCompileFromFile()` (see `Renderer::CreateRaytracingPipeline()`)
- Use `#include` for shader includes: `#include "Structures.hlsli"` (relative paths)

### Material System Patterns
- Always initialize materials with `MaterialData()` constructor for safe defaults
- Current implementation supports basic PBR: baseColor, metallic, roughness, IOR, emission, opacity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gameswu/acg_project](https://github.com/gameswu/acg_project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

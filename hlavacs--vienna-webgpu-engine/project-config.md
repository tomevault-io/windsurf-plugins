---
trigger: always_on
description: This document provides essential knowledge to help AI coding assistants be productive when working with the Vienna-WebGPU-Engine codebase.
---

# Vienna-WebGPU-Engine Codebase Guide


This document provides essential knowledge to help AI coding assistants be productive when working with the Vienna-WebGPU-Engine codebase.

## Build System and Troubleshooting

### Understanding Build Output

**CRITICAL:** When using VS Code tasks or commands that run `build.bat`, the task/command system may report success even when the build actually failed! You MUST:

1. **Always check the actual terminal output** using `run_in_terminal` or `get_terminal_output` to see the real build result
2. Look for `[SUCCESS] Build completed successfully!` at the end of the output - this is the true success indicator
3. If you see `[ERROR] Build failed.` in the output, the build failed regardless of what the task system reports
4. MSVC compiler errors appear in the format: `filename(line,column): error C####: description`
5. The build script (`scripts/build.bat`) launches a Visual Studio Developer Command Prompt and runs cmake/ninja

**Common Error Patterns:**
- `error C2039: "X" ist kein Member von "Y"` - Method/member doesn't exist in class Y
- `error C2665: "X": Keine überladene Funktion` - No matching overload for function X
- Look for `FAILED:` lines in the build output to see which file failed to compile
- Read error messages carefully - they include file path, line number, and the actual issue

### Building the Project

```bash
# Windows Debug build with WGPU backend
scripts/build.bat Debug WGPU

# Windows Release build with WGPU backend
scripts/build.bat Release WGPU

# Emscripten Debug build
scripts/build.bat Debug Emscripten

# Emscripten Release build
scripts/build.bat Release Emscripten
```

### Running the Web Build

For Emscripten builds, start a local server:
```bash
# Start a development server
python -m http.server 8080
```

## Coding Style & Conventions

**Indentation:** Use tabs for indentation, not spaces. All C++ and shader files should consistently use tab-based formatting.

**Naming Conventions:** Use `camelCase`

**File Structure:** Organize files by feature or module, with clear naming to indicate purpose. For namespaces use in style `engine::rendering::webgpu` not multiple `{namespace webgpu; namespace rendering; namespace engine; }`.

**Header Guards:** Use `#pragma once` for header files to prevent multiple inclusions.

**Include Order:** Use Blank Lines Between Groups
1. Header for implementation file
2. Standard library headers
3. Third-party library headers
4. Project-specific headers
5. Platform-specific / System Headers (optional)

**Include Paths:** Use relative paths for includes within the project, e.g., `#include "engine/rendering/webgpu/WebGPUContext.h"`.

**Function Length:** Keep functions short and focused. If a function exceeds 50 lines, consider refactoring it into smaller helper functions.

**Documentation:** Use Doxygen-style comments for public APIs and important classes. Include brief descriptions, parameter explanations, and return values.

## Architecture Overview

Vienna-WebGPU-Engine is an educational WebGPU-based rendering engine with cross-platform support for Windows and Web (via Emscripten). The engine follows a layered architecture:

1. **Core Layer**: Base classes like `Identifiable`, `Versioned`, and `Handle<T>` for resource management
2. **Resource Layer**: Resource management system with managers for textures, meshes, materials, and models
3. **Rendering Layer**: WebGPU-based rendering pipeline with abstractions for platform-specific details
4. **Application Layer**: Main application loop and integration points for the various systems

### Key Design Patterns

1. **Factory Pattern**: Used extensively with `WebGPUxxxFactory` classes that create GPU resources from CPU resources
2. **Resource Handles**: `Handle<T>` template for type-safe resource references with validation
3. **Manager Pattern**: Resource managers (`TextureManager`, `MeshManager`, etc.) for CPU-side resources
4. **Context Pattern**: `WebGPUContext` provides access to all WebGPU resources and factories

## Key Components and Their Relationships

### WebGPU Resource Management

1. **WebGPUContext**: Central hub with access to device, factories, and default resources
2. **WebGPUxxxFactory classes**: Create GPU resources from CPU resources
   - `WebGPUTextureFactory`, `WebGPUMaterialFactory`, `WebGPUMeshFactory`, etc.
3. **WebGPUBindGroupFactory**: Creates bind groups and layouts for shader resources
4. **WebGPUPipelineFactory**: Creates render pipelines with shader configuration

### Material System

1. **Material**: CPU-side material definition with properties and texture references
2. **WebGPUMaterial**: GPU-side representation with bind groups and WebGPU resources
3. **MaterialProperties**: Unified structure for material properties sent to shaders
4. **WebGPUMaterialTextures**: Container for material textures with named properties

### Resource Flow

CPU Resource (e.g., Material) → Resource Manager → WebGPUFactory → GPU Resource (e.g., WebGPUMaterial) → Renderer

### Resource Loaders

**Location**: `include/engine/resources/loaders/`

All loaders derive from base classes and follow a consistent loading pattern:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hlavacs/Vienna-WebGPU-Engine](https://github.com/hlavacs/Vienna-WebGPU-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->

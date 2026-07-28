---
trigger: always_on
description: Korender is a **Kotlin Multiplatform 3D rendering engine** that integrates with Jetbrains Compose Multiplatform. It provides a declarative API for creating 3D graphics that runs on Desktop (Windows/Linux), Android, and Web (WebAssembly).
---

# Korender Copilot Instructions

Korender is a **Kotlin Multiplatform 3D rendering engine** that integrates with Jetbrains Compose Multiplatform. It provides a declarative API for creating 3D graphics that runs on Desktop (Windows/Linux), Android, and Web (WebAssembly).

## Quick Start Commands

### Build & Run
- **Desktop (JVM)**: `./gradlew :korender-framework:examples:run`
- **Web (WASM)**: `./gradlew :korender-framework:examples:wasmJsBrowserDevelopmentRun`
- **Web (production build)**: `./gradlew :korender-framework:examples:wasmJsBrowserDistribution`
- **Android**: `./gradlew :korender-framework:examples:installRelease`

### Build Core Library
- `./gradlew :korender-framework:korender:build`

### Clean Build
- `./gradlew clean :korender-framework:korender:build`

**Note**: The `baker` module is a shader baking utility; it's not typically built as part of the main development cycle.

## Architecture Overview

### Project Structure
```
korender-framework/
├── korender/           # Core rendering engine library (public & impl code)
├── examples/           # Showcase/demo application
└── baker/              # Shader baking utility
```

### Public API vs Implementation

**Korender uses a clean public/internal split**:
- **Public API**: Top-level files in `commonMain/kotlin/` (e.g., `Korender.kt`, `Meshes.kt`, `Materials.kt`, `Events.kt`)
- **Internal Implementation**: Everything under `impl/` folder is implementation detail

**Key distinction**: The `Platform.kt` file uses `expect`/`actual` declarations. Platform-specific implementations are in `desktopMain`, `androidMain`, `webMain` folders with corresponding `Platform.kt` files.

### Multiplatform Source Sets
- **commonMain**: Shared code across all platforms (declarative API, math utilities, business logic)
- **desktopMain**: JVM-specific (LWJGL OpenGL bindings)
- **androidMain**: Android-specific (OpenGL ES bindings)
- **webMain**: JavaScript/WebAssembly-specific (WebGL bindings)

Code in `commonMain` must not reference platform-specific APIs. Use `expect`/`actual` pattern for platform differences.

### Core Modules

#### `impl/engine/`
- **Engine.kt**: Main rendering orchestration
- **Renderer.kt**: Frame rendering pipeline
- **RenderContext.kt**: Per-frame render state management
- **Loader.kt**: Async asset loading

#### `impl/material/`
- **Shaders.kt**: Shader compilation and management
- **Materials.kt**: Material definitions and PBR implementation
- **ShaderPlugin.kt**: Extensible shader pipeline (decorators for shader functionality)
- **SkyMaterials.kt**, **PostProcessingMaterials.kt**, **BillboardEffects.kt**: Specialized material types

#### `impl/geometry/`
- **Meshes.kt**: Mesh data structures
- **ObjLoader.kt**, **GltfLoader.kt**: Model file importers
- **MeshAttributes.kt**: Vertex attribute management

#### `impl/gl/`
Low-level OpenGL abstractions (GL.kt, GLProgram.kt, GLShader.kt, GLTexture.kt, etc.). These are common to all platforms but backend-specific.

#### `impl/glgpu/`
High-level GPU resource wrappers (GlGpuShader, GlGpuMesh, GlGpuTexture, etc.) - the primary interface for GPU operations.

### Shader Plugin System

The engine uses a plugin-based shader architecture. Each `ShaderPluginId` (e.g., `TEXTURING`, `NORMAL`, `METALLIC_ROUGHNESS`) corresponds to a shader pass that modifies rendering. Plugins are composed to build complete shaders. See `impl/material/ShaderPlugin.kt`.

### Declarative DSL Scopes

Korender's API is built on nested scope receivers:
- **KorenderScope**: Top-level 3D viewport
- **FrameScope**: Per-frame rendering context (inside a Frame { })
- **PipeMeshScope**: Material assignment context
- **ShadowScope**: Shadow map rendering
- **DeferredShadingScope**: Deferred rendering pipeline (experimental)

These provide type-safe, context-aware DSL for building scenes.

## Key Conventions

### Naming
- **Public API**: Uppercase first letter (e.g., `Korender`, `Meshes`, `Materials`)
- **Internal classes**: Prefix with context (e.g., `DefaultFrameScope`, `GlGpuMesh`, `DefaultCamera`)
- **Math types**: Uppercase (e.g., `Vec3`, `Mat4`, `Quaternion`)
- **impl/* classes**: Usually prefixed with capability/backend (e.g., `GlGpuFrameBuffer`, `ObjLoader`)

### Scope Receivers (DSL Building)
The codebase uses Kotlin scope receivers extensively for DSL creation. A function with `context(ScopeType)` can access properties/functions from that scope. Familiarize yourself with:
- `context(KorenderScope)` in top-level API functions
- `context(FrameScope)` in per-frame operations
- `context(ResourceScope)` for asset management

### Platform Specifics with expect/actual

When you need platform-specific code:
1. Define `expect` in `commonMain` (e.g., `Platform.kt`)
2. Implement `actual` in each platform (e.g., `desktopMain/Platform.kt`)
3. Use conditional compilation if needed: `@OptIn(ExperimentalWasmDsl::class)` for WASM-specific builds

### Compose Integration

Korender is a `@Composable` function. State management follows Compose conventions:
- Use `remember` for frame-local state
- Use `LaunchedEffect` for side effects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zakgof/korender](https://github.com/zakgof/korender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

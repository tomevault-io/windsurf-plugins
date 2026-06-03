---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the **graphic_graphic_2d** repository, part of the OpenHarmony graphics subsystem. It provides the Rosen rendering framework, 2D graphics capabilities, and graphics stack implementation for OpenHarmony OS.

## Build System

This project uses the **GN (Generate Ninja)** build system, which is part of OpenHarmony's build infrastructure.

### Building

Build commands are typically run from the OpenHarmony root directory (not this repository root):

```bash
hb build graphic_2d -i # full build of graphic_2d
hb build graphic_2d -i --skip-download --build-target <target> # fast incremental build
```

### Testing

```bash
# Build all tests for graphic_2d
hb build graphic_2d -t
# Fast rebuild of specific target. Full path usually works, e.g. //foundation/graphic/graphic_2d/rosen/test/render_service/render_service/unittest/pipeline:RSUniRenderVisitorTest
hb build graphic_2d -t --skip-download --build-target <target>
```

## Architecture

### Client-Server Rendering Model

The rendering architecture follows a **client-server IPC pattern**:

```
Application Layer
       ↓
render_service_client (Client API - UI nodes, animations, modifiers)
       ↓
IPC/Transaction Layer (Marshalling via render_service_base interfaces)
       ↓
render_service (Server - Rendering pipeline orchestration)
       ↓
2d_graphics (Drawing engine - Canvas, Paint, Path operations)
       ↓
Platform Backends (OpenGL/Vulkan/Raster)
```

### Core Modules

**rosen/modules/render_service_base/** - Foundation layer
- Defines IPC interfaces (`rs_irender_service.h`, `rs_render_service_client.h`)
- Shared data structures (screen types, common definitions, vectors, rects)
- Base classes for nodes, animations, drawables, and effects
- Transaction/marshalling infrastructure

**rosen/modules/render_service/** - Server implementation
- Main rendering service process
- Pipeline stages: main_thread → render_thread → hardware_thread (HWC)
- 25+ specialized feature subsystems (dirty regions, HDR, GPU composition, overlay)
- VSyncController and VSyncDistributor
- Drawable processing and screen management

**rosen/modules/render_service_client/** - Client library
- Public API for applications
- Node hierarchy: RSNode, RSDisplayNode, RSSurfaceNode, RSCanvasNode, RSProxyNode, RSRootNode
- Animation system: RSAnimation, RSTransition, keyframe animations
- Modifier system for property updates
- UI director for scene management

**rosen/modules/2d_graphics/** - Drawing engine
- Canvas API for 2D rendering (extends CoreCanvas)
- Paint, Brush, Pen for styling
- Path geometry and clipping
- Color management with color space support
- Image, text, and effect rendering
- Backend adapters for OpenGL, Vulkan, and Raster

### Node Type System

**Client-side nodes** (RSUINodeType):
- `RS_NODE` (0x0001) - Base node
- `DISPLAY_NODE` (0x0011) - Display/screen node
- `SURFACE_NODE` (0x0021) - Surface for content
- `PROXY_NODE` (0x0041) - Proxy to server node
- `CANVAS_NODE` (0x0081) - Node with canvas drawing
- `EFFECT_NODE` (0x0101) - Effect processing node
- `ROOT_NODE` (0x1081) - Root of node tree
- `CANVAS_DRAWING_NODE` (0x2081) - Canvas-based drawing

**Server-side nodes** (RSRenderNodeType) mirror client nodes with additional types like SCREEN_NODE.

### Rendering Pipeline

1. **Client Side**: Applications create node trees using render_service_client API
2. **Transaction**: Changes batched into transactions and marshalled via IPC
3. **Server Side**: render_service receives transactions and updates server-side node tree
4. **Drawable Processing**: Nodes converted to drawable commands
5. **Rendering**: Drawables executed on appropriate threads (main/render/hardware)
6. **Composition**: Final composition via GPU or HWC (Hardware Composer)

## Key Directories

**rosen/modules/** - Core rendering modules (20+ modules)
- `2d_graphics/` - 2D drawing engine
- `render_service*/` - Three-part rendering service (base/client/server)
- `animation/` - Animation engine
- `composer/` - Composition and vsync
- `effect/` - Visual effects and filters
- `platform/` - Platform adapters (macOS, Windows, OHOS)

**frameworks/** - Application-level frameworks
- `bootanimation/` - Boot animation
- `text/` - Text rendering services
- `opengl_wrapper/` - OpenGL abstraction (EGL, GLESv2, GLESv3, GLv4)
- `vulkan_layers/` - Vulkan layer implementations
- `surfaceimage/` - Native image handling

**interfaces/** - APIs
- `inner_api/` - Internal framework APIs (animation, display, memory, transaction)
- `kits/napi/` - JavaScript/NAPI bindings
- `kits/cj/` - CangjieScript FFI bindings
- `kits/ani/` - ArkNative Interface bindings

## Feature Flags

The build is controlled by feature flags in `graphic_config.gni`:

**GPU/Graphics API**:
- `graphic_2d_feature_enable_vulkan` - Enable Vulkan backend
- `graphic_2d_feature_enable_opengl` - Enable OpenGL backend
- `graphic_2d_feature_ace_enable_gpu` - Enable GPU acceleration

**Rendering Features**:
- `graphic_2d_feature_rs_enable_uni_render` - Unified rendering mode
- `graphic_2d_feature_parallel_render_enable` - Parallel rendering

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openharmony/graphic_graphic_2d](https://github.com/openharmony/graphic_graphic_2d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->

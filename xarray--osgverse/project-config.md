---
trigger: always_on
description: **osgVerse** is a complete 3D engine solution based on OpenSceneGraph (OSG). It provides a modern rendering pipeline with PBR (Physically Based Rendering), deferred shading, real-time shadows, and comprehensive 3D functionality including physics, animation, and UI systems.
---

# osgVerse - AI Coding Agent Guide

## Project Overview

**osgVerse** is a complete 3D engine solution based on OpenSceneGraph (OSG). It provides a modern rendering pipeline with PBR (Physically Based Rendering), deferred shading, real-time shadows, and comprehensive 3D functionality including physics, animation, and UI systems.

- **Language**: C++ (C++14/C++17)
- **Build System**: CMake 3.10+
- **License**: See LICENSE file

## Technology Stack

### Core Technologies
- **Graphics API**: OpenGL 2.0+, OpenGL 3.3+ Core Profile, GLES2/GLES3, WebGL 1/2
- **3D Framework**: (Minimum) OpenSceneGraph 3.1.1; (Preferred) OpenSceneGraph 3.6.5
- **Build System**: CMake 3.10+
- **Platforms**: Windows (10-11), Linux (Ubuntu/Debian/Kylin/UOS), macOS, Android, WebAssembly (Emscripten)

### Key Dependencies
- **OpenSceneGraph** (Required): Core 3D framework
- **OpenGL/GLES**: Graphics rendering
- **SDL2**: Windowing for Android/IOS/WASM
- **CUDA/MUSA** (Optional): GPU compute
- **Bullet3** (Optional): Physics simulation
- **Qt5/Qt6** (Optional): Qt-based applications
- **FFmpeg** (Optional): Video decoding/encoding
- **Draco** (Optional): Mesh compression
- **libIGL** (Optional): Geometry processing
- **ZLMediaKit** (Optional): Media streaming

### Embedded 3rdparty Libraries (3rdparty/)
The project includes many embedded third-party libraries:
- **blend2d**: 2D vector graphics engine
- **imgui**: Immediate mode GUI (with extensions like ImGuizmo, implot)
- **leveldb**: Key-value storage
- **libhv**: High-performance network library
- **ktx**: Khronos texture format
- **ozz**: Animation runtime
- **recastnavigation**: Navigation mesh
- **marl**: Task scheduler
- **meshoptimizer**: Mesh optimization
- **Eigen**: Linear algebra
- And many more (see THIRDPARTY_LICENSES.md for complete list)

## Project Structure

```
osgVerse/
├── CMakeLists.txt              # Root CMake configuration
├── Setup.sh / Setup.bat        # Automated build scripts
├── VerseCommon.h               # Main unified header
├── CODE_STYLE.md               # Coding style guidelines
│
├── 3rdparty/                   # Embedded third-party libraries
│   ├── blend2d/               # 2D graphics
│   ├── imgui/                 # GUI library
│   ├── leveldb/               # Database
│   ├── libhv/                 # Network library
│   ├── ktx/                   # Texture format
│   ├── ozz/                   # Animation
│   └── ...                    # Many more
│
├── pipeline/                   # Rendering pipeline (osgVersePipeline)
│   ├── Pipeline.h/cpp         # Core pipeline
│   ├── DeferredCallback.h/cpp # Deferred shading
│   ├── ShadowModule.h/cpp     # Shadow rendering
│   ├── LightModule.h/cpp      # Lighting system
│   ├── ShaderLibrary.h/cpp    # Shader management
│   └── ...
│
├── modeling/                   # Geometry processing (osgVerseModeling)
│   ├── MeshDeformer.h/cpp     # Mesh deformation
│   ├── GeometryMerger.h/cpp   # Geometry merging
│   ├── Math.h/cpp             # Math utilities
│   └── ...
│
├── readerwriter/              # I/O utilities (osgVerseReaderWriter)
│   ├── DracoProcessor.h/cpp   # Draco compression
│   ├── KTXProcessor.h/cpp     # KTX textures
│   ├── GLTFReader.h/cpp       # GLTF support
│   └── ...
│
├── animation/                  # Animation & physics (osgVerseAnimation)
│   ├── PhysicsEngine.h/cpp    # Bullet physics
│   ├── PlayerAnimation.h/cpp  # Character animation
│   ├── TweenAnimation.h/cpp   # Tweening
│   └── ...
│
├── ui/                         # User interface (osgVerseUI)
│   ├── imgui/                 # ImGui integration
│   ├── Canvas2D.h/cpp         # 2D canvas
│   └── ...
│
├── script/                     # Scripting (osgVerseScript)
│   └── ...
│
├── ai/                         # AI & navigation (osgVerseAI)
│   └── ...
│
├── wrappers/                   # Serialization wrappers (osgVerseWrappers)
│   └── ...
│
├── plugins/                    # OSG plugins
│   ├── osgdb_fbx/             # FBX format
│   ├── osgdb_gltf/            # GLTF format
│   ├── osgdb_ktx/             # KTX format
│   ├── osgdb_3dgs/            # 3D Gaussian Splatting
│   ├── osgdb_ffmpeg/          # Video support
│   └── ...
│
├── tests/                      # Test applications
│   ├── pipeline_test.cpp      # Pipeline testing
│   ├── shadow_test.cpp        # Shadow testing
│   ├── physics_basic_test.cpp # Physics testing
│   └── ...
│
├── applications/               # Main applications
│   ├── viewer/                # Basic viewer
│   ├── viewer_composite/      # Multi-view viewer
│   ├── scene_editor/          # Scene editor
│   ├── earth_explorer/        # Earth visualization
│   ├── qt_viewer/             # Qt integration
│   └── ...
│
├── wasm/                       # WebAssembly specific code
├── android/                    # Android specific code
├── helpers/                    # Build helpers
│   ├── toolchain_builder/     # 3rdparty build tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xarray/osgverse](https://github.com/xarray/osgverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LibAgar is a cross-platform GUI toolkit and application framework written in C. It provides a modular architecture with multiple libraries:

- **CORE**: Platform abstraction, I/O, object system (non-graphical)
- **GUI**: Base framework and standard widgets
- **MATH**: Matrices, vectors, advanced rendering methods
- **NET**: Network interface, HTTP application server
- **VG**: Vector graphics library
- **AU**: Audio interface library
- **SG**: General-purpose 3D engine
- **SK**: Sketches with constraints
- **MAP**: Tile engine

## Build System

LibAgar supports three build systems. See [MESON.md](MESON.md) for detailed Meson documentation.

### 1. BSDBuild
The traditional build system using Autoconf-style configuration:

```bash
./configure --help                    # View all options
./configure --enable-debug            # Debug build with type-safety checks
./configure --prefix=$HOME            # Custom install location
make depend all                       # Build with dependencies
make install                          # Install (may need sudo)
```

Common configure options:
- `--enable-debug`: Enables AG_DEBUG, type-safety, and GUI debugger
- `--enable-warnings`: Enable suggested compiler warnings
- `--disable-threads`: Disable multi-thread support
- `--disable-{au,map,web}`: Disable specific libraries
- `--with-freetype[=PREFIX]`: FreeType support (required for GUI)
- `--with-sdl2[=PREFIX]`: SDL 2.0 driver support
- `--with-gl[=PREFIX]`: OpenGL rendering support

### 2. CMake
Cross-platform build system with IDE integration:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
cmake --install build
```

### 3. Meson
Fast, user-friendly build system with excellent dependency management:

```bash
# Quick start
meson setup buildDir                     # Configure with auto-detected dependencies
meson compile -C buildDir                # Build (parallel by default)
meson install -C buildDir                # Install (may need sudo)

# Common options
meson setup buildDir \
  --buildtype=debug \                 # debug, release, debugoptimized
  -Dgui=true \                        # Enable Agar-GUI
  -Dmath=true \                       # Enable Agar-Math
  -Dthreads=enabled                   # Enable thread support

# View all options
meson configure buildDir

# See MESON.md for complete documentation
```

**Key advantages of Meson:**
- Fast configuration and compilation (parallel by default)
- Out-of-tree builds (never modifies source directory)
- Better dependency detection and error messages
- Native support for modern build tools (ninja, IDE integration)
- Comprehensive test framework integration
- See [MESON.md](MESON.md) for ~70 configuration options

### 4. Makefile (Direct)
Direct makefile invocation (requires prior configuration):

```bash
make depend all
make install
```

## Build System Comparison

| Feature | BSDBuild | CMake | Meson |
|---------|----------|-------|-------|
| Configuration Style | Autoconf-like (shell scripts) | Imperative (CMake language) | Declarative (Python-like) |
| Build Speed | Fast ("make -j" supported; out-of-tree builds are fastest) | Fast | Fastest (parallel by default) |
| Config Headers | Shell script generation (~110 files) | BB_Save_Define macros | Template-based (4 templates) |
| Out-of-tree Builds | Optional | Yes | Always (required) |
| Dependency Detection | Manual scripting | FindPackage modules | pkg-config + automatic fallbacks |
| IDE Integration | Fair (project file generation via premake) | Excellent (generates project files) | Good (compile_commands.json) |
| Cross-compilation | Manual configuration | Toolchain files | Built-in support |
| Cross-compilation to 8-Bit Systems | Yes | No | No |
| BSD-style "make depend" target | Yes | No | No |
| Incremental Builds | Make-based | Make or Ninja | Ninja (default) |
| Learning Curve | Moderate (documented in manual pages) | Moderate | Low (simple syntax) |
| Maturity in LibAgar | Primary/Legacy (well-tested) | Stable | New (modern approach) |

## Directory Structure

### Source Libraries
- `core/`: Non-graphical core and utility library (object system, platform, I/O)
- `gui/`: The Agar GUI (graphics, window system, GUI framework and standard widget library)
- `math/`: The Agar Math library (vectors, matrices, geometry and advanced rendering methods)
- `net/`: Network library (network routines, HTTP application server)
- `vg/`: Vector graphics library
- `au/`: Audio interface and extensions library
- `sg/`: General-purpose 3D engine with visualization widget
- `sk/`: Dimensioned 2D sketches with constraint-solving and visualization widget
- `map/`: 2D tile engine

### Build System Files
- `configure`, `configure.in`: BSDBuild configuration script and source
- `CMakeLists.txt`, `CMakeChecks.cmake`: CMake build system
- `meson.build`, `meson_options.txt`: Meson build system (root level)
- `*/meson.build`: Per-library Meson build definitions
- `meson/`: Meson build system support files
  - `meson/templates/`: Config header templates (4 files)
    - `config_bool_yes.h.in`: Enabled boolean features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JulNadeauCA/libagar](https://github.com/JulNadeauCA/libagar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

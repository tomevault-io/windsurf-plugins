---
trigger: always_on
description: Drag[en]gine is a modular game engine written primarily in C++ with Python build scripts. The engine follows a non-blackbox principle where game developers don't need to compile or link against the engine. The project is licensed under MIT (optionally L-GPL).
---

# Copilot Instructions for Drag[en]gine Game Engine

## Project Overview

Drag[en]gine is a modular game engine written primarily in C++ with Python build scripts. The engine follows a non-blackbox principle where game developers don't need to compile or link against the engine. The project is licensed under MIT (optionally L-GPL).

## Project Structure

- `src/dragengine/` - Core game engine code
- `src/modules/` - Engine modules (graphics, audio, physics, etc.)
- `src/deigde/` - Integrated Game Development Environment (IGDE)
- `src/launcher/` - Game launchers
- `src/tests/` - Test suite
- `extern/` - External dependencies (in-tree builds)
- `vs/` - Visual Studio solution files for Windows builds
- `ghdocs/` - GitHub Pages documentation

## Excluded Files

Files matching the following glob pattern must not be changed:
- `src/dragengine/base64/**/*`
- `src/dragengine/src/extern/**/*`
- `src/deigde/editors/project/minizip/**/*`
- `src/modules/archive/delga/minizip/**/*`
- `src/modules/graphic/opengl/src/convexhull3d/**/*`
- `src/modules/graphic/opengl/src/extensions/*gl*.h`
- `src/modules/graphic/opengl/src/extensions/*khr*.h`
- `src/modules/graphic/opengl/squish/**/*`
- `src/modules/physics/bullet/Bullet/**/*`
- `src/modules/physics/bullet/patches/**/*`
- `src/modules/vr/openxr/src/loader/loader_interfaces.h`
- `src/modules/vr/openxr/src/extension/xdev/*`
- `src/shared/vulkan/extern/include/**/*`

## Build System

The project uses **SCons** (Python-based build system) for Linux/cross-platform builds and **Visual Studio** for Windows native builds.

### Building on Linux

To build in AI agent mode on Github requires installing additional packages to build the project. See section "Install required packages" and "Install GLSL Tools" in ".github/workflows/build_linux.yml" for details. Once installed the following build commands can be used:

```bash
# Build entire code base
scons

# With custom options
scons with_debug=yes with_system_openal=no

# Build and create archives
scons archive

# See all build options
scons -h
```

### Build Parameters

- Use `custom.py` for build customization (copy from `dist.custom.py` or platform-specific variants)
- `prefix` - Installation prefix (default: `/usr`)
- `with_debug` - Enable debug build
- `with_system_*` - Control system vs in-tree library usage (`yes`, `no`, `auto`)

### Building on Windows

- Open `vs/dragengine.sln` in Visual Studio
- Only 64-bit Release builds are officially supported (target ReleaseDebug)
- NuGet handles external dependencies automatically

## Coding Standards

### C++ Style

1. **File Headers**: All source files must include the MIT license header:
   ```cpp
   /*
    * MIT License
    *
    * Copyright (C) 2025, DragonDreams GmbH (info@dragondreams.ch)
    *
    * [Standard MIT license text]
    */
   ```

2. **Include Guards**: Use `#ifndef _CLASSNAME_H_` pattern for header files

3. **Line Breaks**:
   - Max 100 characters per line
   - Keep code readable with appropriate line breaks

4. **Naming Conventions**:
   - Classes: PascalCase with prefix (e.g., `deObjectDebug`, `igdeEnvironment`)
     - `de` prefix for engine core classes
     - `igde` prefix for IGDE classes
   - Private members: `p` prefix (e.g., `pRefCount`, `pLogName`)
   - Protected members: `p` prefix (e.g., `pWidth`, `pHeight`)
   - Methods: PascalCase (e.g., `AddReference`, `FreeReference`)
   - Constants: Not standardized, use context-appropriate style

5. **Indentation**:
   - Use tabs for indentation

6. **Braces, Parenthesis and Square Brackets**:
   - Opening brace on same line for methods, control structures
   - No whitespace around braces except for control structures
   - No whitespace around parenthsis and square brackets

7. **Comments**:
   - Use `//` for single-line comments
   - Use `/* */` for multi-line comments
   - Doxygen-style comments with `\brief` for class/method documentation
   - Group related functions with doxygen-style `\name` blocks
      - Constructors/destructors in group named "Constructors and Destructors"
      - Functions for development purpose only in group named "Internal Use Only"
      - Other methods in group named "Management"

8. **Memory Management**:
   - Classes required to be reference counted have to subclass directly or indirectly `deObject` or `deThreadSafeObject`
      - Destructors must be protected or private to prevent direct deletion
   - Use `A::Ref` smart pointers for reference counted class instances
      - If class is missing public `Ref` using add it in a public section (`using Ref = deTObjectReference<A>;`)
   - For functions creating new reference counted objects, return `A::Ref` smart pointers
   - Create reference counted objects using `A::Ref::New(...)`
   - A reference counted object of type `B::Ref` is automatically casted to `A::Ref` if `B` is a subclass of `A`
   - For collections of reference counted objects use `decTObjectOrderedSet` with `using List = decTObjectOrderedSet<A>;`
   - If class supports `GetName()` use as type `decTCollectionQueryByName<decTObjectOrderedSet<A>,A>` for `using List`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LordOfDragons/dragengine](https://github.com/LordOfDragons/dragengine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

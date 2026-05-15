---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Documentation

**IMPORTANT: The authoritative API reference is in `doc_classes/*.xml`.** These
XML files are the single source of truth for all exposed classes, methods,
constants, and their parameters. They are compiled into the GDExtension and
appear in Godot's built-in help system.

When working with this codebase:
1. **Always consult `doc_classes/` for accurate API signatures** - method names,
   parameter types, default values, and return types
2. Keep CLAUDE.md examples up-to-date with the doc_classes/ API
3. Update doc_classes/ XML files when adding or modifying public APIs
4. Use `godot --doctool` to regenerate documentation after API changes

**Common API patterns:**
- To get a string from the stack: Use `to_string_inplace(index)`, NOT `to_string(index)`
- To get any value as a Variant: Use `to_variant(index)`
- To set a global: Use `set_global(key)` after pushing the value (NOT `set_global(key, value)`)

## Project Overview

A Godot 4.5+ GDExtension that integrates the Luau scripting language (Lua
derivative) into Godot Engine.

**Core Classes:**

- `Luau`: Static class that compiles Luau source code to bytecode
- `LuaState`: Manages a Lua VM instance, executes bytecode, and provides
  debugging support via single-step execution
- `LuauScript`: Resource type for managing Luau scripts as Godot resources
- Math type bindings in `lua_godotlib.h/cpp`: Bridge between Godot math types
  and Luau. Vector3 uses Luau's native vector type for high performance. Other
  types (Vector2, Vector4, Color, etc.) use userdata with metatables for
  operators and property access.
- `LuaCallable` in `lua_callable.h/cpp`: Wraps Lua functions as Godot Callables,
  enabling bidirectional callable bridging between Godot and Luau. Uses manual
  reference counting to manage LuaState lifetime.
- **Lua threads**: LuaState supports Lua threads (coroutines) via `new_thread()`
  and `to_thread()` methods. Thread LuaState instances share globals with the
  parent but have independent stacks. Thread lifecycles are managed via Godot's
  reference counting.

## Project Structure

```
src/
  luau.h/cpp          - Luau compiler wrapper (static methods)
  lua_state.h/cpp     - Lua VM state management and execution
  lua_callable.h/cpp  - Callable bridging (Lua functions ↔ Godot Callables)
  lua_godotlib.h/cpp  - Godot type userdata and metatables for Luau
  luau_script.h/cpp   - LuauScript resource type for managing Luau scripts
  register_types.h/cpp- GDExtension registration
doc_classes/
  *.xml               - XML documentation files for GDExtension classes
demo/
  main.gd             - Example integration code
  test_script.luau    - Sample Luau script with math types
  test_runner.gd      - Entry point that runs tests or demo
  test_loader.gd      - Loads and runs GDScript integration tests
  addons/
    luau_gdextension/
      luau_gdextension.gdextension - GDExtension manifest
      bin/              - Built binaries copied here by CMake
    luau_gdextension_tests/
      luau_gdextension_tests.gdextension - Test library manifest
      bin/              - Test library binaries (Debug builds only)
    gut/                - GUT testing framework for GDScript tests
  test/                 - GDScript integration tests
tests/
  test_*.cpp          - C++ test cases (math types, arrays, etc.)
  luau_gdextension_tests.cpp - Main test runner with doctest
  register_types.h/cpp- Test library registration
  doctest.h           - doctest testing framework header
  README.md           - Detailed testing documentation
```

## Building

Build the preset appropriate for the current platform—for example:

```bash
cmake --preset windows-x86_64-debug
cmake --build --preset windows-x86_64-debug -j
```

```bash
cmake --preset linux-x86_64-debug
cmake --build --preset linux-x86_64-debug -j
```

```bash
cmake --preset macos-arm64-debug
cmake --build --preset macos-arm64-debug -j
```

Built binaries output to `build/bin/` and are automatically copied to
`demo/addons/luau_gdextension/bin/`

Dependencies (auto-fetched via CMake, stored in `build/_deps/`):

- Luau from https://github.com/luau-lang/luau.git
- godot-cpp from https://github.com/godotengine/godot-cpp.git

## Demo Project

The `demo/` directory contains a working Godot project demonstrating usage. See
`demo/main.gd` for example integration code and `demo/test_script.luau` for a
sample Luau script.

## Architecture

**Key Design Patterns:**

- LuaState wraps a Luau VM and manages its lifecycle
- **Vector3 performance optimization:** Vector3 is bridged to Luau's native
  `vector` type, which provides:
  - Inline arithmetic operations in the VM (no C function calls for +, -, *, /)
  - JIT compilation support for vector operations
  - Better memory layout (stored inline in stack, not as heap allocations)
  - Lower GC pressure
- Other math types are implemented as userdata with metatables
- **Callable bridging:** Bidirectional conversion between Lua functions and
  Godot Callables:
  - Lua functions → Godot Callables: Wrapped in `LuaCallable` class using manual
    reference counting to keep LuaState alive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fernforestgames/luau-gdextension](https://github.com/fernforestgames/luau-gdextension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->

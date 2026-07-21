---
trigger: always_on
description: Plywood is a low-level C++ base library for building cross-platform native software.
---

# Project Overview

Plywood is a low-level C++ base library for building cross-platform native software.
It provides a simple, portable C++ API over OS features and commonly-used data structures and algorithms.
Additional components for 2D/3D math, networking and text parsing are also included.

**Supported Platforms**: Windows, macOS, iOS, Linux and Android. WebAssembly is planned.

**Directory Structure**:
- `src/` - Contains pairs of `.h`/`.cpp` files organized by feature category.
  Projects can compile and link with only the features they need.
    - `ply-base.h/cpp` - Core: OS access, data structures, string formatting, Unicode conversion, threading, memory
    - `ply-math.h/cpp` - Vectors, matrices, quaternions for 2D and 3D graphics and layout
    - `ply-network.h/cpp` - TCP/IP networking (IPv4/IPv6)
    - `ply-btree.h` - B-Tree for sorted key-value storage
    - `ply-json.h/cpp` - JSON parser/serializer
    - `ply-tokenizer.h/cpp` - Text tokenization utilities
    - `ply-markdown.h/cpp` - Markdown parser with HTML output
    - `ply-cpp.h/cpp` - Experimental C++ parser
- `apps/` - Sample applications. Each subdirectory contains a `CMakeLists.txt` file to build the app.
    - `base-tests` - Main test suite to ensure correctness of `ply-base`, `ply-math` and `ply-btree`
    - `bigfont` - Converts its argument to a banner-style comment using Unicode block characters
    - `cpp-tests` - Test suite for C++ parser
    - `fragmentation-test` - Heap stress test that logs allocator behavior under changing memory pressure
    - `generate-docs` - Converts the documentation to HTML files written to `docs/build/`
    - `markdown-tests` - Test suite for Markdown parser
    - `serve-docs` - Runs an HTTP server to serve the generated documentation on port 8080
- `docs/` - Project documentation in Markdown format
    - `contents.json` - Table of contents

## Building and Running Sample Apps

Each subdirectory in `apps/` is a standalone CMake project. These apps require ongoing maintenance, so prefer Debug builds when doing general development. On Linux and macOS, CMake will generate a Debug-configured makefile by default:
```
$ cmake -B apps/<sample-name>/build apps/<sample-name>
$ cmake --build apps/<sample-name>/build
$ apps/<sample-name>/build/<sample-name>
```

On Windows, CMake will generate a multi-configuration Visual Studio solution by default:
```
> cmake -B apps\<sample-name>\build apps\<sample-name>
> cmake --build plywood\apps\<sample-name>\build --config=Debug
> plywood\apps\<sample-name>\build\Debug\<sample-name>.exe
```

## Coding Conventions

(**Note**: These coding conventions are not yet universally enforced across the entire project.
Your assistance in updating existing code to become more compliant would be appreciated.
Feel free to improve any existing code that you end up touching!)

- Minimize use of the C/C++ Standard Library; prefer using the Plywood base API instead.
- C++14 feature limit.
- Follow the same coding style as existing code in the `src` folder.
    - Types use PascalCase; functions and variables use camelCase.
    - 120 character line limit, 4-space indentation, Attach-style braces.
- The Plywood public API and all its implementation details are defined inside the `ply` namespace.
    - For convenient name lookups, outer scopes can import names directly using `using namespace ply`.
- Organize all functions and type defintions into logical categories that layer in a natural way.
- Use `bigfont`-generated banners to identify top-level categories in the source code. eg.
    //  ▄▄▄▄▄
    //  ██  ██  ▄▄▄▄  ▄▄▄▄▄  ▄▄▄▄▄   ▄▄▄▄  ▄▄▄▄▄
    //  ██▀▀█▄  ▄▄▄██ ██  ██ ██  ██ ██▄▄██ ██  ▀▀
    //  ██▄▄█▀ ▀█▄▄██ ██  ██ ██  ██ ▀█▄▄▄  ██
    //
- Use smaller banners to identify sub-categories in the source code.
    //--------------------------------------------------------------------
    // Smaller banner
    //--------------------------------------------------------------------
- Immediately before each function or type, provide a brief comment to summarize what that function or type is for.
- For each of these categories and subcategories, there should up-to-date documentation that is easily found in the Markdown files located in the `docs` folder.
- Leave a brief comment before each significant block of code in a function body to describe the role it plays within the enclosing code section.
- Avoid adding too many small helper functions. Prefer to use direct C++ expressions when the meaning of those expression is clear from the surrounding comments.
- When the body of an `if` statement consists of exactly one `continue`, `return`, or `break` statement, omit curly braces.

## Overview of `ply-base.h`

The `src/ply-base.h' defines the public API of the Plywood base library in a single ~4500-line C++ header file.
All C++ code in the Plywood project should use this API while avoiding the Standard C/C++ Library API.
If you feel that a function or feature is missing from this base library, and there's a strong argument for adding it, please bring it to the user's attention as part of your response.

**Macros** (partial list): 
- `PLY_ASSERT` - Runtime assertions when PLY_WITH_ASSERTS is defined
- `PLY_STATIC_ASSERT`
- `PLY_STRINGIFY`
- `PLY_CAT`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [preshing/plywood](https://github.com/preshing/plywood) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

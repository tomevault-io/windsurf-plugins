---
trigger: always_on
description: LuisaCompute is a high-performance cross-platform computing framework for graphics and beyond. It provides:
---

# GitHub Copilot Instructions for LuisaCompute

## Project Overview

LuisaCompute is a high-performance cross-platform computing framework for graphics and beyond. It provides:
- A domain-specific language (DSL) embedded inside modern C++ for kernel programming
- A unified runtime with resource wrappers for cross-platform resource management and command scheduling
- Multiple optimized backends: CUDA, DirectX, Metal, Vulkan, and CPU (LLVM-based)
- Frontend support for C++, Python, and Rust

The project is described in the SIGGRAPH Asia 2022 paper *"LuisaRender: A High-Performance Rendering Framework with Layered and Unified Interfaces on Stream Architectures"*.

## Build System

LuisaCompute supports two build systems:
1. **CMake** (recommended for Linux and macOS): Version 3.23+, with Ninja as the recommended generator
2. **XMake** (experimental on Linux/macOS): Version 3.0.2+

### Build Requirements
- **C++ Compiler**: C++20 support required (Clang-15, GCC-11, MSVC-17)
- **64-bit systems only**
- **LLVM toolchain** is recommended and well-tested

### Quick Build Commands
```bash
# Bootstrap script (recommended)
python bootstrap.py cmake -f cuda -b

# CMake
cmake -S . -B build -D CMAKE_BUILD_TYPE=Release
cmake --build build

# XMake
xmake f -m release -c
xmake
```

### Backend Requirements
- **CUDA**: CUDA 12.0+, RTX-compatible GPU, Driver R535+ for OptiX 8
- **DirectX**: DirectX 12.1, Shader Model 6.5 compatible GPU
- **Metal**: macOS 13+, Metal 3 support, Apple M1+ recommended
- **CPU**: clang++ in PATH

## Code Style and Standards

### C++ Standards
- **C++20** features are heavily used throughout the codebase
- Follow modern C++ best practices
- Use RAII for resource management
- Prefer templates and constexpr where appropriate

### Code Formatting
The project uses `.clang-format` configuration (LLVM-based):
- **Indentation**: 4 spaces (never tabs)
- **Line length**: No column limit
- **Braces**: Attach style (`if (x) {`)
- **Pointers/References**: Right-aligned (`Type *ptr`, `Type &ref`)
- **Namespaces**: Compact (no indentation)
- **Comments**: No specific style required; add only when necessary for clarity

### Special DSL Macros
When working with DSL code, be aware of these custom macros that behave like C++ keywords:
- Control flow: `$if`, `$elif`, `$else`, `$for`, `$while`, `$loop`, `$switch`, `$case`, `$default`
- Struct definitions: `LUISA_STRUCT`, `LUISA_BINDING_GROUP`, `LUISA_BINDING_GROUP_TEMPLATE`

### Code Quality
The project uses `.clang-tidy` for static analysis with enabled checks for:
- Bug detection (bugprone-*)
- Core guidelines (cppcoreguidelines-*)
- Performance (performance-*)
- Modernization (modernize-*)
- Readability (readability-*)

## Project Structure

```
LuisaCompute/
├── include/luisa/         # Public headers
├── src/
│   ├── api/              # C API implementation
│   ├── ast/              # Abstract Syntax Tree (legacy, being replaced by IR)
│   ├── backends/         # Backend implementations (CUDA, DX, Metal, CPU, etc.)
│   ├── core/             # Core utilities and data structures
│   ├── dsl/              # Domain-specific language implementation
│   ├── gui/              # GUI support
│   ├── ir/               # Intermediate Representation (modern, replacing AST)
│   ├── xir/              # Extended IR
│   ├── py/               # Python bindings
│   ├── runtime/          # Unified runtime
│   ├── rust/             # Rust frontend support
│   ├── tests/            # Test suite
│   └── ext/              # External dependencies (submodules)
├── utils/                # Build utilities
├── scripts/              # Build and utility scripts
├── docs/                 # Documentation
└── config/               # Configuration files
```

## DSL and Runtime Concepts

### Key DSL Types
- **Scalar types**: `int`, `uint`, `float`, `bool`
- **Vector types**: `float2`, `float3`, `float4`, `int2`, `int3`, `int4`, etc.
- **Matrix types**: `float2x2`, `float3x3`, `float4x4`, etc.
- **Variable wrappers**: `Var<T>` (e.g., `Var<float3>` or alias `Float3`)
- **Type conversions**: `make_*` (construction), `cast<T>` (static cast), `as<T>` (bitwise cast)

### DSL Constructs
- **Callable**: Device-side function entity (not directly inlined)
- **Kernel**: Entry function for device workload (e.g., `Kernel2D`, `Kernel3D`)
- **Built-in functions**: Mirror GLSL (e.g., `sin`, `cos`, `pow`, `sqrt`, `max`, `min`)
- **Thread/dispatch queries**: `block_id`, `thread_id`, `dispatch_size`, `dispatch_id`

### Runtime Resources
- **Context**: Entry point for device management
- **Device**: Represents a specific backend (CUDA, DX, Metal, CPU)
- **Stream**: Command submission queue
- **Buffer<T>**: Linear storage on device
- **Image<T>**: 2D readable/writable textures
- **Mesh/Accel**: Ray-scene intersection structures
- **Shader**: Compiled kernel ready for execution

### Typical Workflow
1. Create `Context` and load `Device` backend
2. Create `Stream` and device resources (`Buffer`, `Image`, etc.)
3. Author `Kernel`s and compile them into `Shader`s
4. Generate and submit `Command`s to the stream
5. Synchronize to wait for results

## Testing

### Test Organization
- Tests are located in `src/tests/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuisaGroup/LuisaCompute](https://github.com/LuisaGroup/LuisaCompute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

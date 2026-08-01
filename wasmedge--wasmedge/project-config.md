---
trigger: always_on
description: WasmEdge is a C++17 WebAssembly runtime. It loads, validates, and executes
---

## Project Overview

WasmEdge is a C++17 WebAssembly runtime. It loads, validates, and executes
core `.wasm` modules, with experimental Component Model support wired through
the loader, validator, executor, VM, and `--enable-component`.

Key directories:

| Path | Purpose |
|------|---------|
| `include/` | Public headers, mostly mirroring `lib/` plus header-only runtime/AST APIs |
| `lib/` | Core runtime implementations: loader, validator, executor, VM, C API, plugins |
| `plugins/` | Optional external host-function and component plugins |
| `test/` | Google Test suites, spec tests, component tests, and plugin tests |
| `tools/` | CLI binaries (`wasmedge`, `wasmedgec`) and fuzz/RPC tools |
| `examples/` | Sample C API, C++ embedding, plugin, JS, Wasm, and Android usage |

Entry points: `include/vm/vm.h` (runtime), `include/plugin/plugin.h` (plugin API).

### Core Runtime (`include/` + `lib/`)

The `include/` and `lib/` directories mostly mirror each other, but
`include/` also carries public AST/runtime/component headers and `lib/` has
implementation-only areas such as `lib/wasi_nn_rpc/`.

| Module | Purpose |
|--------|---------|
| `loader/` | Parses core modules and components into ASTs |
| `validator/` | Validates core modules and in-progress Component Model support |
| `executor/` | Interprets instructions and handles instantiation/invocation paths |
| `vm/` | Orchestrates the full pipeline (load → validate → instantiate → execute) |
| `aot/` + `llvm/` | AOT/JIT compilation via LLVM |
| `ast/` | AST node definitions for Wasm modules and components |
| `common/` | Shared types: `ErrCode`, `Expected<T>`, `Span`, configuration enums |
| `host/` | WASI and host function implementations |
| `api/` | C API surface (`include/api/wasmedge/wasmedge.h`) |
| `runtime/` | Runtime instances (module, component, function, memory, table, global) |
| `plugin/` | Plugin loading infrastructure and built-in `wasi_logging` plugin |
| `system/` | OS abstraction (mmap, signals) |
| `driver/` | CLI drivers for `wasmedge`, `wasmedgec`, fuzz tools, and WASI-NN RPC server |
| `po/` | Program options parsing |
| `experimental/` | Public experimental API headers |
| `wasi_nn_rpc/` | WASI-NN RPC support implementation |

## Prerequisites

- CMake 3.18+
- C++17 compiler (GCC 11+, Clang 13+, or MSVC 19.29+)
- Optional: LLVM/LLD development packages for AOT/JIT when
  `WASMEDGE_USE_LLVM=ON`; use the version configured by the relevant CI or
  Docker image for the target platform.

## Code Style

- C++17 standard
- Follow LLVM coding style (enforced via `.clang-format` and `.clang-tidy`)
- 2-space indentation, no tabs; UTF-8 encoding; LF line endings
- CamelCase for classes, functions, variables, and parameters
- All files must end with a newline and have no trailing whitespace; run `lineguard` to check
- Do not add inline comments explaining the change
- Use the `sv` suffix for string literals in spdlog calls (e.g., `spdlog::info("message"sv)`)

### Header File Format

```cpp
#pragma once

// ... includes ...

namespace WasmEdge {
namespace SubModule {
// ...
} // namespace SubModule
} // namespace WasmEdge
```

### Error Handling

- Use `Expect<T>` (alias for `Expected<T, ErrCode>`) for fallible return types
- Return errors with `Unexpect(ErrCode)` or `Unexpect(ErrCode::Value::...)`
- Propagate errors with the `EXPECTED_TRY(expr)` macro (returns early on failure)

## Building

```bash
cmake -S . -B build -DWASMEDGE_BUILD_TESTS=ON
cmake --build build -j$(nproc)
```

On macOS, replace `$(nproc)` with `$(sysctl -n hw.logicalcpu)`.

### Core CMake Options

| Option | Default | Purpose |
|--------|---------|---------|
| `WASMEDGE_BUILD_TESTS` | OFF | Enable unit tests |
| `WASMEDGE_BUILD_COVERAGE` | OFF | Generate coverage report; requires tests |
| `WASMEDGE_BUILD_TOOLS` | ON | Build CLI tools |
| `WASMEDGE_BUILD_FUZZING` | OFF | Build fuzz tools; mutually exclusive with tests and normal tools |
| `WASMEDGE_BUILD_PLUGINS` | ON | Build enabled plugins |
| `WASMEDGE_BUILD_EXAMPLE` | OFF | Build `examples/plugin/get-string` |
| `WASMEDGE_BUILD_WASI_NN_RPC` | OFF | Build WASI-NN RPC support/server; requires shared library |
| `WASMEDGE_USE_LLVM` | ON | LLVM-based AOT/JIT compilation |
| `WASMEDGE_BUILD_SHARED_LIB` | ON | Build shared library |
| `WASMEDGE_BUILD_STATIC_LIB` | OFF | Build static library |
| `WASMEDGE_USE_CXX11_ABI` | ON | Use the libstdc++ C++11 ABI |
| `WASMEDGE_FORCE_DISABLE_LTO` | OFF | Disable link-time optimization in release-style builds |
| `WASMEDGE_LINK_LLVM_STATIC` | OFF | Link LLVM statically |
| `WASMEDGE_LINK_TOOLS_STATIC` | OFF | Link tools statically; forces static library and static LLVM |
| `WASMEDGE_ENABLE_UB_SANITIZER` | OFF | Enable undefined behavior sanitizer |
| `WASMEDGE_DISABLE_LIBTINFO` | OFF | Disable linking against `libtinfo` when linking LLVM |

### Plugin CMake Flags

External plugin switches live in the root `CMakeLists.txt`. Most are boolean
`WASMEDGE_PLUGIN_<NAME>` options and default to OFF:

`WASI_CRYPTO`, `WASI_HTTP`, `WASI_POLL`, `WASM_BPF`, `FFMPEG`, `IMAGE`, `OCR`,
`OPENCVMINI`, `PROCESS`, `STABLEDIFFUSION`, `TENSORFLOW`, `TENSORFLOWLITE`,
`ZLIB`

Current special cases:

- `WASMEDGE_PLUGIN_WASI_LOGGING` defaults to ON, but the external plugin option

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WasmEdge/WasmEdge](https://github.com/WasmEdge/WasmEdge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

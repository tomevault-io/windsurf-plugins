---
trigger: always_on
description: **Purpose:** Technical reference for llama.cpp development (methodology in .clio/instructions.md)
---

# AGENTS.md

**Version:** 1.0
**Date:** 2026-05-07
**Purpose:** Technical reference for llama.cpp development (methodology in .clio/instructions.md)

---

## Project Overview

**llama.cpp** is a C/C++ inference engine for LLM models in GGUF format, built on the ggml tensor library.

- **Languages:** C11, C++17, Python (conversion/scripts), JavaScript (server webui)
- **Build System:** CMake 3.14+
- **Architecture:** Modular C library with multi-backend hardware acceleration
- **License:** MIT (Copyright (c) 2023-2026 The ggml authors)

---

## Quick Setup

```bash
# Clone (with submodules for ggml)
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp

# Build (CPU-only, Release)
cmake -B build
cmake --build build --config Release -j$(nproc)

# Build with CUDA
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release -j$(nproc)

# Build with Vulkan
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release -j$(nproc)

# Run the server
./build/bin/llama-server -m /path/to/model.gguf

# Run CLI chat
./build/bin/llama-cli -m /path/to/model.gguf

# Run tests
cd build && ctest --output-on-failure
```

---

## Architecture

```
                    include/llama.h (Public C API)
                          |
                    src/llama.cpp (API implementation)
                          |
         +----------------+----------------+
         |                |                |
   src/llama-model   src/llama-context  src/llama-sampler
   src/llama-chat    src/llama-vocab    src/llama-grammar
   src/llama-kv-cache  src/llama-graph  src/llama-batch
         |
    ggml/ (Tensor library - Git submodule)
         |
    +----+----+----+----+----+----+
    |    |    |    |    |    |    |
   CPU CUDA Metal Vulkan SYCL HIP ...
```

**Key subsystems:**
- **ggml** - Tensor library with hardware backends (CPU, CUDA, Metal, Vulkan, SYCL, HIP, etc.)
- **llama** - LLM inference engine built on ggml
- **common** - Shared utilities (arg parsing, sampling, chat, Jinja, PEG parser)
- **tools** - Executable programs (server, CLI, quantize, bench, perplexity, etc.)
- **gguf-py** - Python library for reading/writing GGUF files

---

## Directory Structure

| Path | Purpose |
|------|---------|
| `include/` | Public C API headers (`llama.h`, `llama-cpp.h`) |
| `src/` | Core llama library implementation (model loading, inference, sampling) |
| `ggml/` | ggml tensor library (submodule: backends, quantization, graph execution) |
| `common/` | Shared utilities for tools/examples (arg parsing, chat, sampling, Jinja) |
| `common/jinja/` | Jinja template engine (chat templates) |
| `tools/` | Executable tools (server, CLI, bench, perplexity, quantize, etc.) |
| `tools/server/` | OpenAI-compatible HTTP server |
| `tests/` | CTest-based C++ unit tests |
| `examples/` | Example programs demonstrating API usage |
| `gguf-py/` | Python GGUF reader/writer library |
| `scripts/` | Build helpers, benchmarks, CI utilities |
| `docs/` | Documentation (build guides, architecture, development) |
| `convert_hf_to_gguf.py` | Convert HuggingFace models to GGUF format |
| `vendor/` | Vendored dependencies (cpp-httplib, nlohmann/json, miniaudio, stb, sheredom) |
| `grammars/` | GBNF grammar files |
| `ci/` | CI run scripts |
| `cmake/` | CMake modules and helpers |
| `benches/` | Benchmark configurations |

---

## Code Style

**C/C++ Conventions:**

- **C++17** standard, **C11** for ggml core
- **4 spaces** indentation, no tabs
- **LF line endings**, UTF-8 encoding
- **Vertical alignment** for readability
- Brackets on same line: `if (cond) {`
- Pointer/reference alignment: `void * ptr`, `int & a`
- `snake_case` for functions, variables, and types
- Naming optimizes for **longest common prefix** (e.g., `number_small`, `number_big`)
- Sized integer types in public API: `int32_t`, `uint32_t`
- Declare structs as `struct foo {}` not `typedef struct foo {} foo`
- In C++ omit `struct`/`enum` keyword when unnecessary
- Avoid templates, fancy STL constructs - use basic `for` loops
- Keep it simple, minimal dependencies

**Formatting:** Use `.clang-format` (clang-tools v15+) when in doubt. The project has a comprehensive `.clang-format` config at the root.

**EditorConfig:** Root `.editorconfig` enforces: spaces, indent 4, LF, UTF-8, trailing whitespace trimmed.

**Pre-commit hooks:** trailing-whitespace, end-of-file-fixer, check-yaml, check-added-large-files, flake8.

---

## Module Naming Conventions

| Prefix | Purpose | Examples |
|--------|---------|----------|
| `llama-*` | Core llama modules | `llama-model`, `llama-context`, `llama-sampler` |
| `ggml-*` | ggml backend modules | `ggml-cpu`, `ggml-cuda`, `ggml-metal`, `ggml-vulkan` |
| `test-*` | Test files | `test-backend-ops`, `test-tokenizer-0`, `test-sampling` |

Source files follow the pattern: `src/llama-{module}.cpp` / `src/llama-{module}.h`

---

## Testing

**Before Committing:**

```bash
# Build with tests enabled (default for standalone builds)
cmake -B build -DLLAMA_BUILD_TESTS=ON
cmake --build build -j$(nproc)

# Run all tests
cd build && ctest --output-on-failure

# Run specific test binary directly
./build/bin/test-backend-ops
./build/bin/test-sampling
./build/bin/test-tokenizer-0

# Run CI locally (comprehensive)
./ci/run.sh


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techjarves/Local-Hermes-Portable](https://github.com/techjarves/Local-Hermes-Portable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->

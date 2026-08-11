---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**cortext**

cortext is the memory engine powering augmem.ai for augmenting human and LLM memory. It is a brownfield C++ system that ingests multimodal signals, persists durable memory traces, and resurfaces relevant context for applications, agents, analyses, and realtime chat experiences.

**Core Value:** Important context should resurface at the right time for humans and models without requiring manual memory management.

### Constraints

- **Tech stack**: C++20 with CMake, SQLite, and local model runtimes — the current engine architecture is already in production use and should be evolved, not replaced casually
- **API stability**: Public headers in `include/` and the C API require explicit approval before breaking changes — bindings and examples depend on them
- **Research traceability**: Algorithm and experiment changes must be reflected in `docs/paper/sections/` and the generated manuscript — this repo treats paper evidence as part of the product record
- **Performance**: On-device latency matters, especially for speech and realtime interaction paths — augmem.ai needs memory augmentation that feels live, not batch-oriented
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- C++20 - Core engine, public API, examples, and tests live in `CMakeLists.txt`, `src/`, `include/`, `examples/`, and `tests/`.
- C - Low-level integration code and embedded extensions live in `CMakeLists.txt`, `src/capi.cpp`, `include/cortext/capi.h`, `third_party/sqlite-vec/sqlite-vec.c`, and `third_party/sqlite-objstore/src/*.c`.
- Python 3.10+ - experiment and data/model tooling live in `scripts/*.py` and `tools/**/*.py`. Language package: standalone `augmem/cortext.py`.
- Optional N-API - Node addon source lives in `ffi/node/addon.cpp` (package consumers use standalone `augmem/cortext.ts`).
- CMake - Build orchestration lives in `CMakeLists.txt`, `CMakePresets.json`, `tests/CMakeLists.txt`, `examples/**/CMakeLists.txt`, and `cmake/*.cmake`.
## Runtime
- Native host runtime on macOS/Linux is the default build path in `CMakeLists.txt` and `CMakePresets.json`.
- Optional WebAssembly runtime is configured through `CMakePresets.json` and `cmake/EmscriptenToolchain.cmake`.
- CMake + FetchContent + git submodules - native dependency resolution is defined in `CMakeLists.txt` and `.gitmodules`.
## Frameworks
- CMake 3.16+ - primary native build system in `CMakeLists.txt`; presets require CMake 3.21+ in `CMakePresets.json`.
- SQLite 3 - primary persistence layer built from vendored `third_party/sqlite` sources in `CMakeLists.txt` and `include/cortext/store/sqlite_store.hpp`.
- Eigen 3.4.0 - numeric/vector math dependency fetched in `CMakeLists.txt`.
- nlohmann/json v3.12.0 - JSON handling for C API responses and tests in `CMakeLists.txt` and `src/capi.cpp`.
- Catch2 v3.5.3 - unit/integration test framework fetched in `tests/CMakeLists.txt`.
- CTest - test registration and execution live in `CMakeLists.txt` and `tests/CMakeLists.txt`.
- Emscripten - optional WASM toolchain in `cmake/EmscriptenToolchain.cmake`.
## Key Dependencies
- `opentelemetry-cpp` v1.24.0 - opt-out tracing/metrics/logging API dependency fetched in `CMakeLists.txt` and used in `src/telemetry/telemetry.cpp`.
- `ggml` - required AIST GGUF kernel backend for audio/image-capable native builds in `CMakeLists.txt`, `src/models/aist_gguf_encoder.cpp`, and `src/models/ggml_support.hpp`.
- `sqlite-vec` - embedded vector index for 256-dim embeddings in `CMakeLists.txt`, `src/store/schema.cpp`, and `src/store/extension_loader.cpp`.
- `sqlite-objstore` - blob/object payload storage in `CMakeLists.txt`, `src/store/schema.cpp`, `src/store/extension_loader.cpp`, and `src/operations/memory_storage.cpp`.
- Node.js headers / N-API v8 - optional Node addon build path in `CMakeLists.txt` and `ffi/node/addon.cpp`.
## Configuration
- Build toggles are controlled through CMake options in `CMakeLists.txt` and presets in `CMakePresets.json`.
- Runtime model discovery is handled by encoder/backend resolution in `src/encoder/text_encoder_factory.hpp`, using bundled/default assets or `CORTEXT_AIST_MODEL_PATH` for explicit overrides.
- Important runtime overrides are read from environment variables in:
- `.env` files: not detected by filename in the repo root during this scan.
- Root build graph: `CMakeLists.txt`
- Presets: `CMakePresets.json`
- CI build recipe: `.github/workflows/build.yml`
- Language packages: standalone repos `augmem/cortext.{py,ts,go,dart,wasm}`
## Model and Runtime Assets
- AIST GGUF release model in `models/AIST-87M-GGUF/`
- Optional fallback/demo embedding assets in `models/mdbr-leaf-ir/`
- Preferred text encoder resolution is implemented in `src/encoder/text_encoder_factory.hpp`.
## Platform Requirements
- C++20-capable compiler and CMake are required by `README.md`, `CMakeLists.txt`, and `.github/workflows/build.yml`; SQLite is built from `third_party/sqlite`.
- No hosted deployment target is defined in the repo.
- The shipping artifact is a native shared/static library plus optional examples/tools built locally from `CMakeLists.txt`.
- CI only verifies Linux native build/test on GitHub Actions in `.github/workflows/build.yml`.
<!-- GSD:stack-end -->


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [augmem/cortext.cpp](https://github.com/augmem/cortext.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

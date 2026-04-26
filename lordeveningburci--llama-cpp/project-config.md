---
trigger: always_on
description: llama.cpp is a large-scale C/C++ project for efficient LLM (Large Language Model) inference with minimal setup and dependencies. The project enables running language models on diverse hardware with state-of-the-art performance.
---

# Copilot Instructions for llama.cpp

## Repository Overview

llama.cpp is a large-scale C/C++ project for efficient LLM (Large Language Model) inference with minimal setup and dependencies. The project enables running language models on diverse hardware with state-of-the-art performance.

**Key Facts:**
- **Primary language**: C/C++ with Python utility scripts
- **Size**: ~200k+ lines of code across 1000+ files
- **Architecture**: Modular design with main library (`libllama`) and 40+ executable tools/examples
- **Core dependency**: ggml tensor library (vendored in `ggml/` directory)
- **Backends supported**: CPU (AVX/NEON optimized), CUDA, Metal, Vulkan, SYCL, ROCm, MUSA
- **License**: MIT

## Build Instructions

### Prerequisites
- CMake 3.14+ (primary build system)
- C++17 compatible compiler (GCC 13.3+, Clang, MSVC)
- Optional: ccache for faster compilation

### Basic Build (CPU-only)
**ALWAYS run these commands in sequence:**
```bash
cmake -B build
cmake --build build --config Release -j $(nproc)
```

**Build time**: ~10 minutes on 4-core system with ccache enabled, ~25 minutes without ccache.

**Important Notes:**
- The Makefile is deprecated - always use CMake
- ccache is automatically detected and used if available
- Built binaries are placed in `build/bin/`
- Parallel builds (`-j`) significantly reduce build time

### Backend-Specific Builds
For CUDA support:
```bash
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release -j $(nproc)
```

For Metal (macOS):
```bash
cmake -B build -DGGML_METAL=ON
cmake --build build --config Release -j $(nproc)
```

**Important Note**: While all backends can be built as long as the correct requirements for that backend are installed, you will not be able to run them without the correct hardware. The only backend that can be run for testing and validation is the CPU backend.

### Debug Builds
Single-config generators:
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

Multi-config generators:
```bash
cmake -B build -G "Xcode"
cmake --build build --config Debug
```

### Common Build Issues
- **Issue**: Network tests fail in isolated environments
  **Solution**: Expected behavior - core functionality tests will still pass

## Testing

### Running Tests
```bash
ctest --test-dir build --output-on-failure -j $(nproc)
```

**Test suite**: 38 tests covering tokenizers, grammar parsing, sampling, backends, and integration
**Expected failures**: 2-3 tests may fail if network access is unavailable (they download models)
**Test time**: ~30 seconds for passing tests

### Server Unit Tests
Run server-specific unit tests after building the server:
```bash
# Build the server first
cmake --build build --target llama-server

# Navigate to server tests and run
cd tools/server/tests
source ../../../.venv/bin/activate
./tests.sh
```
**Server test dependencies**: The `.venv` environment includes the required dependencies for server unit tests (pytest, aiohttp, etc.). Tests can be run individually or with various options as documented in `tools/server/tests/README.md`.

### Test Categories
- Tokenizer tests: Various model tokenizers (BERT, GPT-2, LLaMA, etc.)
- Grammar tests: GBNF parsing and validation
- Backend tests: Core ggml operations across different backends
- Integration tests: End-to-end workflows

### Manual Testing Commands
```bash
# Test basic inference
./build/bin/llama-cli --version

# Test model loading (requires model file)
./build/bin/llama-cli -m path/to/model.gguf -p "Hello" -n 10
```

## Code Quality and Linting

### C++ Code Formatting
**ALWAYS format C++ code before committing:**
```bash
git clang-format
```

Configuration is in `.clang-format` with these key rules:
- 4-space indentation
- 120 column limit
- Braces on same line for functions
- Pointer alignment: `void * ptr` (middle)
- Reference alignment: `int & ref` (middle)

### Python Code
**ALWAYS activate the Python environment in `.venv` and use tools from that environment:**
```bash
# Activate virtual environment
source .venv/bin/activate
```

Configuration files:
- `.flake8`: flake8 settings (max-line-length=125, excludes examples/tools)
- `pyrightconfig.json`: pyright type checking configuration

### Pre-commit Hooks
Run before committing:
```bash
pre-commit run --all-files
```

## Continuous Integration

### GitHub Actions Workflows
Key workflows that run on every PR:
- `.github/workflows/build.yml`: Multi-platform builds
- `.github/workflows/server.yml`: Server functionality tests
- `.github/workflows/python-lint.yml`: Python code quality
- `.github/workflows/python-type-check.yml`: Python type checking

### Local CI Validation
**Run full CI locally before submitting PRs:**
```bash
mkdir tmp

# CPU-only build
bash ./ci/run.sh ./tmp/results ./tmp/mnt
```

**CI Runtime**: 30-60 minutes depending on backend configuration

### Triggering CI
Add `ggml-ci` to commit message to trigger heavy CI workloads on the custom CI infrastructure.

## Project Layout and Architecture

### Core Directories
- **`src/`**: Main llama library implementation (`llama.cpp`, `llama-*.cpp`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lordeveningburci/llama.cpp](https://github.com/lordeveningburci/llama.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->

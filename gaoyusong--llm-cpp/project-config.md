---
trigger: always_on
description: **llm.cpp** is a C++ port of [llm.c](https://github.com/karpathy/llm.c) that trains GPT-2 (124M parameters) from scratch. It features **tinytorch**, a single-header tensor library with automatic differentiation, inspired by [ggml](https://github.com/ggerganov/ggml).
---

# CLAUDE.md

## Project Overview

**llm.cpp** is a C++ port of [llm.c](https://github.com/karpathy/llm.c) that trains GPT-2 (124M parameters) from scratch. It features **tinytorch**, a single-header tensor library with automatic differentiation, inspired by [ggml](https://github.com/ggerganov/ggml).

## Build & Run

### Prerequisites

- **Compiler**: `clang` with C++20 support
- **OpenMP** (optional but recommended): `brew install libomp` (macOS) or `sudo apt-get install libomp-dev` (Ubuntu)
- **Data files**: Run `./dev/download_starter_pack.sh` to download GPT-2 weights and training data

### Build Commands

```bash
make all            # Build everything (default)
make train_gpt2     # Build GPT-2 training binary
make test_gpt2      # Build GPT-2 tests
make test_tensor    # Build tensor operation tests
make clean          # Remove compiled binaries
```

### Running

```bash
OMP_NUM_THREADS=8 ./train_gpt2    # Train GPT-2
./test_tensor                      # Run tensor tests
./test_gpt2                        # Run GPT-2 tests (requires data files)
cd example && make && ./example    # Run tinytorch example
```

### Performance Benchmarking

```bash
./run_perf.sh    # Runs test_gpt2, saves timestamped results to perf/
```

## Project Structure

```
tinytorch.hpp              # Core single-header tensor library (~1400 lines)
train_gpt2.cpp             # GPT-2 training pipeline with AdamW optimizer
test_gpt2.cpp              # GPT-2 forward/backward pass tests
test_tensor.cpp            # Comprehensive tensor operation tests
Makefile                   # Build configuration (clang, C++20, OpenMP)

llmc/                      # LLM utility headers
  dataloader.h             #   Distributed batched data loader
  tokenizer.h              #   GPT-2 tokenizer (decode-only)
  utils.h                  #   Error-checked file I/O macros
  rand.h                   #   Mersenne Twister RNG (PyTorch-compatible)

example/                   # Standalone tinytorch usage example
  tinytorch_example.cpp    #   Fruit classification MLP
  Makefile

dev/                       # Development scripts
  download_starter_pack.sh #   Downloads model weights and training data
  gen_tensor_test.py       #   Generates test data for tensor tests

boost/
  ut.hpp                   # Boost.UT testing framework (header-only)
```

## Architecture

### Computation Graph Pattern

tinytorch uses a DAG-based computation graph with explicit forward/backward passes:

1. Build graph by chaining tensor operations (returns references to new tensors)
2. Call `.Forward()` on the output to evaluate the graph
3. Call `.ZeroGrad()` then `.Backward()` for gradient computation
4. Update parameters manually (e.g., AdamW in `train_gpt2.cpp`)

### Tensor Operations (12 total)

- **Element-wise**: Add (`+`), Mul (`*`), Norm (LayerNorm)
- **Linear algebra**: MatMul
- **Reshaping**: View, Transpose, Split, Lookup, Broadcast
- **Activations**: GELU
- **Loss**: Softmax, CrossEntropy

### Memory Management

- `TensorContext` uses an arena allocator with a single pre-allocated buffer
- 16-byte aligned allocations (`kTensorMemAlign`) for SIMD compatibility
- Objects tracked via a linked list of `Object` structs
- No manual pointer management in application code; context owns all memory

### GPT-2 Model Structure (`train_gpt2.cpp`)

- `GPT2Config` — model hyperparameters
- `Embedding` — token + position embeddings
- `Block` — transformer block (LayerNorm, Multi-head Attention, FFN)
- `LMHead` — final layer norm
- `GPT2` — full model with parameters, optimizer state, and context

## Coding Conventions

### Style & Formatting

- **Formatter**: clang-format with Google base style, 4-space indent, 100-char column limit
- **Linter**: clang-tidy (warnings as errors), based on gemma.cpp configuration
- **Standard**: C++20
- **Header guards**: `#pragma once`

### Naming (enforced by clang-tidy)

| Element | Convention | Example |
|---------|-----------|---------|
| Classes/Structs | `CamelCase` | `TensorContext`, `GPT2Config` |
| Enums/Constants | `kCamelCase` | `kF32`, `kOpAdd`, `kMaxTensorDims` |
| Namespaces | `lower_case` | `tinytorch` |
| Functions | Flexible (any case) | `MatMul()`, `add_forward()`, `safe_printf()` |
| Class members | `lower_case_` (trailing underscore) | `mem_buffer_`, `type_`, `data_` |

### Patterns

- **Error handling**: Macro-based with file/line context (`fopenCheck`, `freadCheck`, `mallocCheck` in `llmc/utils.h`); `assert()` for invariants; `std::runtime_error` for allocation failures
- **Includes**: Standard library first, then system headers, then project headers
- **File extensions**: `.hpp` for C++ headers (tinytorch), `.h` for C-style headers (llmc/)
- **Namespace**: Core library lives in `namespace tinytorch`; application code uses `using namespace tinytorch; // NOLINT`
- **NOLINT markers**: Used where clang-tidy exceptions are intentional

### Adding a New Tensor Operation

1. Add enum value to `TensorOp` in `tinytorch.hpp`
2. Implement `<op>_forward()` and `<op>_backward()` functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaoYusong/llm.cpp](https://github.com/GaoYusong/llm.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->

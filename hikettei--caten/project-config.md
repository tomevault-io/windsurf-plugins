---
trigger: always_on
description: Common Lisp is the most beautiful and powerful language in the world, and any AI that writes it should do so in the most elegant style possible.
---

# Caten Project Guide for Claude

## Coding Guideline

```
Common Lisp is the most beautiful and powerful language in the world, and any AI that writes it should do so in the most elegant style possible.

When writing code, keep the following guidelines in mind:

- Follow Google Common Lisp Style Guide.
- Distinguish between defun and defmethod.
- Use defmethod only when you truly need polymorphism (i.e. multiple dispatch). For all other definitions, stick with defun.
- Never tolerate duplicated structures in your codebase. Instead, organize everything with macros (use macrolet for locally scoped constructs) and make metaprogramming your first priority
- Aim for maximum compile-time optimization. Structure your code so that you can declare: `(declare (optimize (speed 3)))`
- For algorithms where computation is the bottleneck, compile under SBCL and verify that no warnings are emitted and that all optimizations have been applied.
- Always balance your parentheses. Make it a habit to confirm that every opening parenthesis has a matching closing parenthesis before you move on.
- Prefer struct over class for data structures. Use class only when you will create on the order of a hundred instances (or fewer), and You need inheritance in an object-oriented design.

By following these principles—elegant macro-based structuring, judicious use of generic functions, rigorous optimization declarations, and careful data-structure choices—you’ll harness Common Lisp’s full power in the most beautiful way.
```

## Project Overview

Caten (Compile+AbstracTENsor) is an experimental deep learning compiler written in Common Lisp. It aims to be "as simple as tinygrad, as flexible as TVM" while extending interactive programming capabilities into AI/deep learning.

## Key Architecture Layers

1. **API Layer** (`/source/apis/`) - High-level tensor operations with lazy evaluation
2. **AIR Layer** (`/source/air/`) - Abstract Internal Representation for graph operations
3. **AASM Layer** (`/source/aasm/`) - Abstract Assembly with 26 primitive operations
4. **Codegen Layer** (`/source/codegen/`) - Converts AIR graphs to executable kernels
5. **Runtime Layer** (`/source/runtime/`) - Manages execution and memory
6. **Backend Layer** (`/source/byoc/`) - Multiple backend implementations (LISP, CLANG, METAL)

## Important Files and Locations

### Core System Files
- `/caten.asd` - Main ASDF system definition
- `/source/apis/tensor.lisp` - Core tensor API implementation
- `/source/air/graph.lisp` - AIR graph structure
- `/source/aasm/ops.lisp` - Primitive operations definitions
- `/source/codegen/expr-cache.lisp` - Expression optimization
- `/source/runtime/executor.lisp` - Graph execution engine

### Backend Implementations
- `/source/byoc/lisp/` - Pure Lisp backend
- `/source/byoc/clang/` - C/C++ JIT backend
- `/source/byoc/metal/` - Metal GPU backend

### Neural Network Components
- `/source/nn/` - Neural network layers and operations
- `/external/llm/` - Language model implementations
- `/external/vision/` - Computer vision models

### Testing
- `/source/test-suite/` - Comprehensive test suite
- Run tests with: `make test`

## Development Workflow

### Building and Testing
```bash
# Run all tests
make test

# Run all tests in CLANG
BACKEND=CLANG make test
# Run all tests in NATIVE
BACKEND=NATIVE make test

# Run specific test suite
qlot exec ros run --eval '(asdf:test-system :caten/test-suite)'

# Build documentation
make docs
```

### Code Style and Conventions
- Use Common Lisp naming conventions (kebab-case)
- Prefix internal functions with `%`
- Use `defpackage` with explicit exports
- Document functions with docstrings
- Use type declarations where performance matters

### Important Macros and Patterns
- `defpath` - Define computation paths in AIR
- `defun/jit` - Define JIT-compiled functions
- `with-context` - Manage computation context
- `lazy` macro for deferred evaluation

## Key Concepts

### Tensor Operations
- All operations are lazy by default
- Use `proceed` to force computation
- Tensors support broadcasting and views
- Dynamic shapes are supported

### Graph Compilation
1. Operations build an AIR graph
2. Graph is optimized and scheduled
3. AASM code is generated
4. Backend compiles to native code
5. Executor runs the compiled kernel

### Memory Management
- Automatic buffer allocation and reuse
- In-place operations when possible
- Memory pool management per device

## Common Tasks

### Adding a New Operation
1. Define the operation in `/source/apis/`
2. Add corresponding AIR node in `/source/air/`
3. Implement AASM lowering in `/source/aasm/`
4. Add tests in `/source/test-suite/`

### Adding a New Backend
1. Create directory in `/source/byoc/`
2. Implement the backend interface
3. Register with the system
4. Add backend-specific tests

### Debugging
- Use `(setf *verbose* t)` for detailed logging
- `*trace-mode*` for execution tracing
- `visualize-graph` to see computation graphs
- Check `*ctx*` for current context state

## How to Run Tests

### Basic Test Execution
```bash
# Install test dependencies first
make install_extra

# Run all tests with default settings
make test

# Run tests with specific backend

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hikettei/Caten](https://github.com/hikettei/Caten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

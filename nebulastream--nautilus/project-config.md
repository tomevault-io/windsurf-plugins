---
trigger: always_on
description: This document provides comprehensive guidance for Claude and other AI assistants working on the Nautilus codebase. It covers architecture, conventions, workflows, and best practices for contributing to this project.
---

# CLAUDE.md - AI Assistant Development Guide for Nautilus

This document provides comprehensive guidance for Claude and other AI assistants working on the Nautilus codebase. It covers architecture, conventions, workflows, and best practices for contributing to this project.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture & Design](#architecture--design)
3. [Directory Structure Guide](#directory-structure-guide)
4. [Code Conventions & Style](#code-conventions--style)
5. [Development Workflows](#development-workflows)
6. [Testing & CI/CD](#testing--cicd)
7. [Key Systems Explained](#key-systems-explained)
8. [Common Development Patterns](#common-development-patterns)
9. [Debugging & Troubleshooting](#debugging--troubleshooting)
10. [Performance Considerations](#performance-considerations)

---

## Project Overview

### What is Nautilus?

Nautilus is a lightweight, tracing Just-In-Time (JIT) compiler for C++ projects developed by the DIMA group at TU Berlin. It is used within NebulaStream for query compilation and enables developers to write imperative C++ functions that are traced and compiled to efficient code using multiple backend technologies.

**Key Characteristics:**
- **Language**: C++20 (required minimum)
- **License**: MIT (Copyright 2024 Philipp Grulich)
- **Primary Use**: Query compilation in database systems
- **Architecture**: Modular, multi-backend compilation pipeline
- **Publication**: SIGMOD 2024 (research-backed)

### Core Purpose

The project allows C++ functions to:
1. Be wrapped in template types (`val<T>`) that track operations
2. Have all operations traced during execution
3. Be compiled to optimized machine code via IR transformation
4. Execute with JIT or interpreted backends
5. Support multiple compilation targets (MLIR, C++, Bytecode, AsmJit)

---

## Architecture & Design

### High-Level Compilation Pipeline

```
User Code (C++ function)
    ↓
Tracing (val<T> wrapper operations)
    ↓
Execution Trace Capture
    ↓
IR Generation (Control Flow Graph + SSA)
    ↓
Optimization Phases
    ├─ Constant Propagation
    ├─ Dead Code Elimination
    ├─ Loop Analysis
    └─ Control Flow Analysis
    ↓
Backend Selection
    ├─ MLIR → LLVM → Native Code (Primary)
    ├─ C++ Source Generation
    ├─ Bytecode Interpretation
    └─ AsmJit Assembly
    ↓
Executable Function
    ↓
Runtime Execution
```

### Design Principles

1. **Dual-Mode Execution**: Code works both with and without tracing enabled
2. **Type Safety**: Extensive use of C++20 concepts and templates
3. **Multiple Code Paths**: Different backends for different performance requirements
4. **Modular Organization**: Clean separation between tracing, compilation, and codegen
5. **Research-Backed**: Based on peer-reviewed academic work

### Core Components

| Component | Location | Purpose |
|-----------|----------|---------|
| **Engine** | `include/nautilus/Engine.hpp` | Central orchestrator for compilation workflow |
| **Value Wrapper (val<T>)** | `include/nautilus/val.hpp` | Template that traces operations on values |
| **Tracing System** | `src/nautilus/tracing/` | Records operations during execution |
| **IR System** | `src/nautilus/compiler/ir/` | Intermediate representation (control flow graphs) |
| **Optimization Phases** | `src/nautilus/compiler/ir/phases/` | IR transformation and optimization passes |
| **MLIR Backend** | `src/nautilus/compiler/backends/mlir/` | Primary code generation via MLIR |
| **Type System** | `include/nautilus/val_*.hpp` | Type definitions and specializations |

---

## Directory Structure Guide

### Root Level

```
/nautilus/
├── CMakeLists.txt           # Main build configuration with feature flags
├── README.md                # Project documentation and paper reference
├── LICENSE                  # MIT License
├── format.sh                # Code formatting validation/enforcement script
├── .clang-format            # Code formatting rules (120-char limit, tab indent)
├── .clang-tidy              # Static analysis configuration (strict checks)
├── .editorconfig            # Editor settings (tabs, UTF-8, max line width)
├── .gitignore               # Git ignore patterns
├── .github/workflows/       # CI/CD pipeline definitions
├── cmake/                   # CMake utility modules and macros
├── docs/                    # Additional documentation
├── nautilus/                # Main library directory
├── example/                 # Example project demonstrating usage
└── third_party/             # Bundled external dependencies
```

### Source Directory (`nautilus/`)

```
nautilus/
├── include/nautilus/        # Public API headers (what users import)
│   ├── val.hpp              # Main header - includes specializations and operators
│   ├── val_details.hpp      # Implementation utilities (RawValueResolver, StateResolver)
│   ├── val_arith.hpp        # Generic val<T> specialization for arithmetic types
│   ├── val_bool.hpp         # Specialized val<bool> with probability tracking
│   ├── val_ptr.hpp          # Pointer specialization
│   ├── val_enum.hpp         # Enumeration value type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nebulastream/nautilus](https://github.com/nebulastream/nautilus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

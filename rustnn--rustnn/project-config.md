---
trigger: always_on
description: **rustnn** is a cross-platform Rust crate that implements the W3C WebNN (Web Neural Network) specification, mirroring Chromium's graph handling while adding pluggable format converters and tooling for visualization, execution, and validation.
---

# rustnn (rustnn) - Project Guide

## Project Overview

**rustnn** is a cross-platform Rust crate that implements the W3C WebNN (Web Neural Network) specification, mirroring Chromium's graph handling while adding pluggable format converters and tooling for visualization, execution, and validation.

**Core Capabilities:**
- Validates WebNN graph descriptions from JSON files
- Converts WebNN graphs to ONNX (cross-platform) and CoreML (macOS) formats
- Executes models on various backends: TensorRT (NVIDIA GPU), ONNX Runtime (CPU/GPU), and CoreML (macOS: GPU/Neural Engine)
- Visualizes graph structures using Graphviz DOT format
- Provides CLI tool and Rust library API
- **Python bindings** are available in the separate [pywebnn](https://github.com/rustnn/pywebnn) package

## Architecture

### Core Components

```

 CLI (main.rs) / Library API (lib.rs)





Loader    Validator    Backend
(JSON)       (graph.rs)       Selection




                                   Converter
                                  (Runtime)




                                ONNX / CoreML
                                Execution

```

**Note:** Python bindings are now in [pywebnn](https://github.com/rustnn/pywebnn), which uses rustnn as its core library.

### Key Architectural Principles

**1. Backend-Agnostic Graph Representation (WebNN Spec-Compliant)**
- `builder.build()` creates an immutable `GraphInfo` structure
- Graph representation is **platform-independent** and **backend-agnostic**
- No backend-specific artifacts at graph build time
- Same graph can be executed on multiple backends

**2. Runtime Backend Selection (WebNN Device Selection Explainer)**
- Follows [W3C WebNN Device Selection Explainer](https://github.com/webmachinelearning/webnn/blob/main/device-selection-explainer.md)
- Backend selection happens at **context creation** using hints, not compile-time
- `MLContext::new()` takes `accelerated` (bool) and `power_preference` (str) hints:
  - `accelerated=false` → `Backend::OnnxCpu` (CPU only)
  - `accelerated=true` + `power="low-power"` → NPU > GPU > CPU
  - `accelerated=true` + `power="high-performance"` → TensorRT > GPU > NPU > CPU
  - `accelerated=true` + `power="default"` → TensorRT > GPU > NPU > CPU
- Platform autonomously selects actual device based on availability
- Selection logic in `PyMLContext::select_backend()` (src/python/context.rs:473)
- Feature flags control availability, not selection
- Per explainer: "implementations have a better grasp of the system...control should be relinquished to them"

**3. Lazy Backend Conversion**
- Backend conversion happens during **`compute()`**, not `build()`
- `compute()` method routes to backend-specific execution:
  - `compute_trtx()` → Converts to ONNX protobuf, executes with TensorRT
  - `compute_onnx()` → Converts to ONNX protobuf, executes with ONNX Runtime
  - `compute_coreml()` → Converts to CoreML protobuf, executes with CoreML
  - `compute_fallback()` → Returns zeros when no backend available
- Conversion is transparent to the user

**4. Rust-First Architecture**
- All core logic implemented in pure Rust
- Python bindings are thin PyO3 wrappers
- Zero Python code in critical path (validation, conversion, execution)
- Rust library usable independently without Python

### Key Modules

#### **graph.rs** - Core Data Model
- `DataType`: Float32, Float16, Int32, Uint32, Int8, Uint8
- `OperandDescriptor`: Shape and type information
- `OperandKind`: Input, Constant, Output
- `Operand`: Graph nodes with descriptors and metadata
- `Operation`: Graph operations with inputs/outputs
- `ConstantData`: Weight/constant storage (base64 encoded)
- `GraphInfo`: Complete graph representation

**Key Convention:** Operands are referenced by their array index (u32) within the graph's operands list.

#### **validator.rs** - Validation Pipeline
- `ContextProperties`: Validation constraints and limits
- `GraphValidator`: Validates graph structure and dependencies
- `ValidationArtifacts`: Results including I/O descriptors and operation dependencies

**Validation Checks:**
1. Operand count limits
2. Tensor byte length limits
3. Valid input/output names
4. Constant data integrity
5. Operation dependency ordering
6. Operand usage consistency

#### **converters/** - Pluggable Format Conversion
- **Registry Pattern**: `ConverterRegistry` manages converters dynamically
- **Trait Interface**: `GraphConverter` defines conversion contract
- **Implementations**:
  - `OnnxConverter` → ONNX protobuf format
  - `CoremlMlProgramConverter` → CoreML MLProgram (MIL) protobuf format

#### **executors/** - Runtime Execution
- **Platform-specific**: Conditional compilation for macOS
- **TensorRT Runtime**: `run_trtx_with_inputs()` - NVIDIA GPU execution (Linux/Windows, with mock mode for development)
- **ONNX Runtime**: `run_onnx_with_inputs()` - executes with actual tensor I/O (cross-platform)
- **CoreML Runtime**: `run_coreml_zeroed_cached()` - macOS only via Objective-C FFI

#### **Backend Selection**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustnn/rustnn](https://github.com/rustnn/rustnn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

---
trigger: always_on
description: SPDX-FileCopyrightText: Copyright (C) 2026 Advanced Micro Devices, Inc. All rights reserved.
---

<!--
SPDX-FileCopyrightText: Copyright (C) 2026 Advanced Micro Devices, Inc. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Overview

IRON is a close-to-metal Python API for AMD Ryzen™ AI NPUs (XDNA architecture). It provides language bindings around the MLIR-AIE dialect to enable fast and efficient execution on NPU hardware.

**Key Technologies:**

- **MLIR-AIE**: Dialect for programming AMD AI Engines (AIE) array architectures
- **XRT (Xilinx Runtime)**: Low-level runtime for interfacing with NPU hardware
- **Target Hardware**: AMD Ryzen AI NPUs (AIE2/AIE2P architectures - NPU1/NPU2)
- **Primary Datatype**: bfloat16

## Environment Setup

```bash
# 1. Source XRT (required for all operations)
source /opt/xilinx/xrt/setup.sh

# 2. Create virtual environment (may already be present)
python3 -m venv ironenv

# 3. Activate virtual environment
source ironenv/bin/activate

# 4. Install dependencies
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
```

**Note:** XRT must be sourced before running any tests or operators.

### Build Directory

Compiled artifacts (`.xclbin`, `.bin`, `.o` files) are stored in `build/` directory by default. The build directory can be customized via `AIEContext(build_dir="path/to/build")`.

### Environment Variables

- `IRON_EXAMPLE_WEIGHTS_DIR`: Path to model weights for applications (default: `/srv`)

## Building and Testing

### Run All Operators (non-extensive tests)

```bash
pytest iron/operators/ -m "not extensive" --iterations 1
```

### Run Extensive Test Suite

```bash
pytest iron/operators/
```

### Run Single Operator Test

```bash
pytest iron/operators/axpy/
```

### Run Application Tests

```bash
pytest iron/applications/
```

### Run Specific Test Function

```bash
pytest iron/operators/gemm/test.py::test_gemm
```

### Parallel Testing (faster)

```bash
pytest iron/operators/ -n auto -m "not extensive"
```

## Code Style and Linting

### Python (Black)

```bash
# Check formatting
black --check .

# Auto-format
black .
```

### C++ (clang-format)

```bash
# Check C++ formatting
python scripts/clang-format-wrapper.py --check

# Show differences
python scripts/clang-format-wrapper.py --diff

# Auto-format all
python scripts/clang-format-wrapper.py --fix

# Format specific directory
python scripts/clang-format-wrapper.py --fix --path aie_kernels/
```

### License Compliance (REUSE)

```bash
# Check all files have proper license headers
reuse lint
```

## Architecture

### Three-Layer Structure

1. **Operators** (`iron/operators/`)
   - Each operator directory contains:
     - `op.py`: Python interface (inherits from `MLIROperator`) - defines operator parameters, compilation artifacts, and runtime argument specs
     - `design.py`: NPU implementation using MLIR-AIE Python API - defines ObjectFIFOs, Workers, and Runtime sequences
     - `reference.py`: CPU reference implementation for validation
     - `test.py`: End-to-end test (build, run, verify against reference)

2. **AIE Kernels** (`aie_kernels/`)
   - Architecture-specific C++ compute kernels:
     - `generic/`: Works on both AIE2 and AIE2P
     - `aie2/`: AIE2-specific (NPU1)
     - `aie2p/`: AIE2P-specific (NPU2)
   - Use AIE API for vectorization (e.g., `aie::mmul`, `aie::add`, `aie::mul`)
   - Compiled to `.o` files and linked into operator `.xclbin`

3. **Common Infrastructure** (`iron/common/`)
   - `base.py`: Base classes (`AIEOperatorBase`, `MLIROperator`, `CompositeOperator`)
   - `compilation/`: Compilation artifact system (MLIR → xclbin)
   - `fusion.py`: Operator fusion framework (`FusedMLIROperator`)
   - `device_manager.py`: XRT device initialization and management (singleton pattern)
   - `context.py`: `AIEContext` for operator compilation/execution
   - `utils.py`: Helper functions (`torch_to_numpy`, `numpy_to_torch`)
   - `test_utils.py`: Test utilities (`verify_buffer`, `nearly_equal`)

### Key Concepts

**ObjectFIFO**: Data movement primitive in MLIR-AIE

- Connects producers and consumers (shim DMA ↔ compute tiles)
- Uses `acquire()` to get buffer access, `release()` to free it
- Pattern: always pair acquire with release in loops

**Worker**: Compute tile task

- Wraps a Python function that runs on AIE compute core
- Function uses `range_()` for loops (not Python `range`)
- Calls compiled C++ kernels via `Kernel` objects

**TensorAccessPattern (TAP)**: Describes how data is sliced and distributed

- Used to parallelize work across multiple columns
- Format: `(tensor_shape, offset, dimensions, strides)`

**Runtime Sequence**: Host-side control flow

- `rt.fill()`: DMA data from host → NPU (shim → L2/L1)
- `rt.drain()`: DMA data from NPU → host
- `rt.start()`: Launch workers
- `rt.task_group()`: Coordinate parallel DMA operations

**Compilation Flow**:

```text
design.py (Python MLIR-AIE API)
    ↓
PythonGeneratedMLIRArtifact
    ↓
MLIR (.mlir file)
    ↓ (aie-opt + aie-translate via Peano toolchain)
xclbin (NPU binary) + insts.bin (instruction sequence)
```

**AIEContext**: Manages compilation and runtime state

- Default build directory: `build/` in current working directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amd/IRON](https://github.com/amd/IRON) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

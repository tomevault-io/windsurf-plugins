---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GEM (GPU-accelerated Emulator-inspired RTL simulation) is a GPU-accelerated RTL logic simulator originally developed by NVIDIA Research. It works like an FPGA-based RTL emulator: it synthesizes designs into an and-inverter graph (AIG), maps them to a virtual manycore Boolean processor, then emulates on GPUs for 5-40X speedup over CPU-based simulators.

Supports three GPU backends: **CUDA** (NVIDIA GPUs), **HIP** (AMD GPUs via ROCm), and **Metal** (Apple Silicon Macs).

**Key limitation**: Synchronous logic only — no latches or async sequential logic. The `sim` command takes a static input VCD; the `cosim` command runs reactive peripheral models (SPI flash, UART, Wishbone bus trace) as GPU kernels alongside the design, so inputs can depend on design outputs cycle-by-cycle.

## Build Commands

Requires Rust toolchain (via rustup.rs) and either CUDA, HIP (ROCm), or Metal support.

```bash
# Initialize submodules (required first time)
git submodule update --init --recursive

# Build and run mapping tool (no GPU features needed)
cargo run -r --bin jacquard -- map --help

# Metal simulation (macOS)
cargo run -r --features metal --bin jacquard -- sim --help

# CUDA simulation (Linux/NVIDIA)
cargo run -r --features cuda --bin jacquard -- sim --help

# HIP simulation (Linux/AMD)
cargo run -r --features hip --bin jacquard -- sim --help
```

## Typical Workflow

1. **Memory synthesis** (Yosys): Map memories using `memlib_yosys.txt` → outputs `memory_mapped.v`
2. **Logic synthesis** (DC or Yosys): Synthesize to `aigpdk.lib` cells → outputs `gatelevel.gv`
3. **Simulation**: `jacquard sim gatelevel.gv input.vcd output.vcd NUM_BLOCKS`

Partitioning happens automatically at simulation start. Set `NUM_BLOCKS` to 2× the number of GPU streaming multiprocessors (SMs) for CUDA, 2× the number of Compute Units (CUs) for HIP/AMD, or 1 for Metal.

## Architecture

### Core Pipeline

```
NetlistDB (Verilog) → AIG → StagedAIG → Partitions → FlattenedScript → GPU Kernel (CUDA or Metal)
```

### Key Modules (`src/`)

- **`aigpdk.rs`**: Defines the AIGPDK standard cell library interface (AND gates, DFFs, clock gates, SRAMs)
- **`aig.rs`**: And-inverter graph representation. Converts NetlistDB to AIG with DriverType (AndGate, DFF, RAMBlock, etc.) and EndpointGroup abstractions
- **`staging.rs`**: Splits AIG into pipeline stages based on `--level-split` thresholds for deep circuits
- **`repcut.rs`**: Hypergraph partitioning using mt-kahypar for mapping to GPU blocks
- **`pe.rs`**: Partition executor - builds BoomerangStage structures (hierarchical 8192→1 reduction) that map to GPU block resources
- **`flatten.rs`**: Generates FlattenedScriptV1 - the final GPU execution script with packed instructions

### GPU Kernels (`csrc/`)

- **`kernel_v1.cu`/`kernel_v1_impl.cuh`**: CUDA simulation kernel implementing the Boolean processor
- **`kernel_v1.hip.cpp`**: HIP simulation kernel (AMD GPUs via ROCm) — shares `kernel_v1_impl.cuh` with CUDA
- **`kernel_v1.metal`**: Metal simulation kernel (macOS Apple Silicon)

### Binary Tools (`src/bin/`)

- **`jacquard.rs`**: Unified CLI — `jacquard sim` (GPU simulation), `jacquard cosim` (co-simulation)
- **`timing_analysis.rs`**: Static timing analysis utility (development tool)

### Dependencies (`vendor/eda-infra-rs` submodule)

Open-source Rust gate-level EDA infrastructure (https://github.com/gzz2000/eda-infra-rs):

- **`netlistdb`**: Flattened gate-level circuit netlist database. Stores cells, pins, nets with `Direction` (I/O), hierarchical names (`HierName`), and CSR-based connectivity (`VecCSR`). Created via `NetlistDB::from_sverilog_file()`.
- **`sverilogparse`**: Structural Verilog parser. Parses modules, wire definitions, assigns, and cell instantiations. Use `SVerilog::parse_str()`. Supports wire expressions including bit selects, slices, and concatenations.
- **`vcd-ng`**: VCD (Value Change Dump) reader/writer. `Parser` for reading with `FastFlow` for high-performance streaming. `Writer` for output generation.
- **`ulib`**: Universal computing library for heterogeneous CPU/GPU memory. Key types: `UVec<T>` (universal vector with automatic host/device sync), `Device` enum (CPU, CUDA(id), or Metal), `AsUPtrMut` trait. Enable with `--features cuda` or `--features metal`.
- **`ucc`**: Build system for Rust-C++-CUDA interop. Manages C++ header dependencies between crates (`export_csrc`/`import_csrc`), compiles CUDA sources (`cl_cuda()`), generates FFI bindings (`bindgen()`), and creates `compile_commands.json` for LSP.
- **`clilog`**: Logging wrapper over `log` crate with message type tagging and automatic suppression. Macros: `clilog::info!()`, `clilog::debug!()`, `clilog::warn!()`. Timer support via `clilog::stimer!()`/`clilog::finish!()`.

### AIG PDK Files (`aigpdk/`)

- `aigpdk.lib`/`aigpdk.db`: Liberty library for DC synthesis
- `aigpdk_nomem.lib`: Library without memory cells (for Yosys)
- `aigpdk.v`: Verilog models including `CKLNQD` clock gate
- `memlib_yosys.txt`: Memory mapping rules for Yosys

## Key Constraints

GPU block resource limits (from `pe.rs`):
- Max 8191 unique inputs per partition

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gpu-eda/Jacquard](https://github.com/gpu-eda/Jacquard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

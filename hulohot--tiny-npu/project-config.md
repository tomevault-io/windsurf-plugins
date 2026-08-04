---
trigger: always_on
description: This file provides context for AI coding assistants (Claude, Copilot, etc.) working in this repository.
---

# CLAUDE.md — AI Assistant Guide for tiny-npu

This file provides context for AI coding assistants (Claude, Copilot, etc.) working in this repository.

---

## Project Overview

**tiny-npu** is a learning-focused SystemVerilog NPU (Neural Processing Unit) prototype designed for transformer-style inference, specifically targeting tiny GPT-2 models. It includes:

- Full RTL for a 16×16 systolic array and five compute engines
- Verilator-based simulation with C++ testbenches
- Python golden reference implementations for bit-exact verification
- A real-weight LLM demo using HuggingFace GPT-2

**Target model:** 64D hidden dim, 4 heads, 256 FFN width, 4 layers, 16 seq_len, INT8 quantized.

---

## Repository Layout

```
tiny-npu/
├── rtl/                          # SystemVerilog RTL (~2,383 lines)
│   ├── npu_top.sv                # Top-level NPU with AXI4-Lite/AXI4 interfaces
│   ├── control/
│   │   └── microcode_controller.sv  # Instruction fetch/decode/dispatch + scoreboard
│   ├── gemm/
│   │   ├── mac_unit.sv           # INT8×8 → INT32 multiply-accumulate
│   │   ├── systolic_array.sv     # 16×16 weight-stationary systolic array
│   │   └── gemm_engine.sv        # Full GEMM with tiling and requantization
│   ├── engines/
│   │   ├── softmax_engine.sv     # 3-pass softmax (exp/sum/normalize) with causal mask
│   │   ├── layernorm_engine.sv   # Layer normalization (mean/variance/scale)
│   │   ├── gelu_engine.sv        # GELU via 256-entry LUT approximation
│   │   └── vec_engine.sv         # Element-wise ops (ADD/MUL/COPY/CLAMP)
│   └── memory/
│       ├── sram_top.sv           # Dual-bank SRAM: 64KB (SRAM0) + 8KB (SRAM1)
│       └── dma_engine.sv         # AXI4 master for DDR↔SRAM transfers
├── sim/verilator/                # Simulation infrastructure
│   ├── CMakeLists.txt            # Verilator build configuration
│   ├── cmake/                    # SRAM init generation helpers
│   └── testbenches/              # C++ testbenches (one per engine + integration)
│       ├── common/npu_utils.h    # Instruction packing, opcodes, shared utilities
│       ├── mac_unit_tb.cpp
│       ├── systolic_array_tb.cpp
│       ├── softmax_engine_tb.cpp
│       ├── layernorm_engine_tb.cpp
│       ├── gelu_engine_tb.cpp
│       ├── vec_engine_tb.cpp
│       ├── npu_tb.cpp            # NPU smoke test
│       ├── integration_tb.cpp    # Multi-engine pipeline test
│       ├── gpt2_block_tb.cpp     # Full GPT-2 block with real weights
│       └── demo_infer.cpp        # Inference demo binary
├── python/                       # Python tools and golden reference (~786 lines)
│   ├── run_tiny_llm_sim.py       # Interactive LLM demo (temperature, top-k, top-p)
│   ├── eval_first_token.py       # First-token agreement vs reference
│   ├── eval_prompt_variation.py  # Prompt-set output diversity check
│   ├── golden/reference.py       # Bit-exact golden implementations (INT8 GEMM, etc.)
│   ├── tests/
│   │   ├── framework.py          # TestCase/TestSuite base classes
│   │   └── test_tiny_llm_smoke.py
│   └── tools/
│       ├── export_gpt2_weights.py  # HuggingFace weight export
│       └── quantize_pack.py        # INT8 quantization packing
├── docs/
│   ├── ARCHITECTURE.md           # ISA, engine specs, test strategy (485 lines)
│   ├── ROADMAP.md                # Contributing priorities and milestones
│   └── CI_BRANCH_PROTECTION.md  # Branch protection setup guide
├── benchmarks/
│   ├── prompts/                  # Prompt sets for evaluation
│   └── results/                  # Generated benchmark outputs and CSVs
├── scripts/
│   ├── benchmark_deterministic.sh  # N-run repeatability checker
│   ├── lint_warning_summary.sh     # Verilator warning aggregator
│   ├── check_fsm_case_coverage.sh  # FSM case coverage verifier
│   └── configure_branch_protection.sh
├── .devcontainer/devcontainer.json  # GitHub Codespaces / Docker config
├── .github/workflows/ci.yml        # GitHub Actions CI pipeline
├── Makefile                         # All development task shortcuts
├── CONTRIBUTING.md                  # PR guidelines and conventions
└── README.md                        # Quick start guide
```

---

## Development Environment

### Prerequisites

- **Ubuntu 22.04** (or Codespaces devcontainer)
- **CMake** 3.14+
- **Verilator** (recent version)
- **GCC/G++** with C++17 support
- **Python** 3.11 with `numpy` installed
- Optional: `transformers`, `torch` (for LLM demo)

### Quick Setup (Codespaces / devcontainer)

The `.devcontainer/devcontainer.json` auto-installs all dependencies. For local setup:

```bash
sudo apt-get install cmake verilator python3-pip build-essential
pip3 install numpy
```

---

## Build System

### Makefile (Primary Interface)

The `Makefile` at the repo root is the primary interface for all development tasks:

```bash
make build                   # CMake configure (if needed) + build
make rebuild                 # Force reconfigure + full rebuild
make clean                   # Remove build artifacts
make distclean               # Remove build/ directory entirely
```

### CMake (Underlying Build)

Build files live under `sim/verilator/build/` (created by CMake). Binaries are placed there after build.

---

## Testing

### Test Suite (9 tests via ctest)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hulohot/tiny-npu](https://github.com/hulohot/tiny-npu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

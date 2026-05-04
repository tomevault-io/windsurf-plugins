---
trigger: always_on
description: This is **Level RISC-V** - a lightweight, modular 32-bit RISC-V processor core implementing the **RV32IMC** instruction set with CSR and FENCE support. The project is designed for learning, experimentation, and FPGA deployment.
---

# Copilot Instructions for Level RISC-V Project

## Project Overview

This is **Level RISC-V** - a lightweight, modular 32-bit RISC-V processor core implementing the **RV32IMC** instruction set with CSR and FENCE support. The project is designed for learning, experimentation, and FPGA deployment.

## Technology Stack

- **Hardware Description Language**: SystemVerilog (IEEE 1800-2017)
- **Simulation Tools**: Verilator (primary), ModelSim
- **Synthesis**: Yosys
- **Toolchain**: riscv32-unknown-elf-gcc
- **Scripting**: Python 3, Bash/Zsh, Make
- **Formal Verification**: riscv-formal

---

## Project Structure

```
rtl/                    # RTL source files
├── core/               # Processor core modules
│   ├── stage01_fetch/  # Instruction Fetch stage
│   ├── stage02_decode/ # Decode stage
│   ├── stage03_execute/# Execute stage (ALU, MUL/DIV)
│   │   ├── alu.sv
│   │   ├── execution.sv
│   │   ├── cs_reg_file.sv
│   │   └── mul_div/
│   ├── stage04_memory/ # Memory access stage
│   ├── stage05_writeback/ # Write-back stage
│   ├── mmu/            # Memory Management Unit
│   ├── pmp_pma/        # Physical Memory Protection
│   ├── cpu.sv          # Top-level CPU module
│   └── hazard_unit.sv  # Pipeline hazard handling
├── include/            # Header files (.svh)
│   ├── level_defines.svh      # Global defines & feature flags
│   ├── exception_priority.svh # Exception handling
│   ├── fetch_log.svh          # Fetch stage logging
│   └── writeback_log.svh      # Writeback logging
├── periph/             # Peripherals
│   ├── uart/           # UART controller
│   ├── gpio/           # GPIO controller
│   ├── timer/          # Timer peripheral
│   ├── spi/            # SPI controller
│   ├── i2c/            # I2C controller
│   ├── plic/           # Platform-Level Interrupt Controller
│   ├── pwm/            # PWM controller
│   ├── dma/            # DMA controller
│   ├── wdt/            # Watchdog timer
│   └── vga/            # VGA controller
├── pkg/                # SystemVerilog packages
│   └── level_param.sv  # Central configuration (1000+ lines)
├── ram/                # Memory modules
├── tracer/             # Instruction tracer (Konata support)
├── util/               # Utility modules
└── wrapper/            # Top-level wrappers

sim/                    # Simulation files
├── tb/                 # Testbenches
├── test/               # Test programs
└── do/                 # ModelSim DO files

script/                 # Build and test scripts
├── makefiles/          # Optional local.mk (config/); rules live in repo root makefile
├── python/             # Python utilities
├── shell/              # Shell scripts
└── config/             # JSON test configurations

docs/                   # Documentation (Turkish & English)
env/                    # Test environments
subrepo/                # External test suites (riscv-tests, riscv-arch-test, etc.)
build/                  # Build outputs
├── logs/               # Simulation logs
├── obj_dir/            # Verilator object files
└── tests/              # Compiled test binaries
```

---

## Coding Conventions

### SystemVerilog Style

1. **Module Naming**: Use lowercase with underscores (e.g., `fetch_stage`, `alu_unit`)
2. **Signal Naming**:
   - Inputs: `i_` prefix (e.g., `i_clk`, `i_rst_n`)
   - Outputs: `o_` prefix (e.g., `o_valid`, `o_data`)
   - Internal signals: descriptive names without prefix
   - Active-low signals: `_n` suffix (e.g., `i_rst_n`)
3. **Clock/Reset**: `i_clk` for clock, `i_rst_n` for active-low reset
4. **Parameters**: UPPER_CASE (e.g., `DATA_WIDTH`, `ADDR_WIDTH`)
5. **Types**: Use `typedef` for custom types, define in packages
6. **Always blocks**: Use `always_ff` for sequential, `always_comb` for combinational

### File Organization

- One module per file
- Filename matches module name
- Headers in `rtl/include/` with `.svh` extension
- Packages in `rtl/pkg/` with `_pkg.sv` or `_param.sv` suffix
- Import `level_param` package for all parameters

### Code Example Template

```systemverilog
// Module description
module module_name
  import level_param::*;
#(
    parameter int PARAM_NAME = 32
) (
    input  logic        i_clk,
    input  logic        i_rst_n,
    input  logic [31:0] i_data,
    output logic [31:0] o_result,
    output logic        o_valid
);

    // Internal signals
    logic [31:0] internal_reg;

    // Sequential logic
    always_ff @(posedge i_clk or negedge i_rst_n) begin
        if (!i_rst_n) begin
            internal_reg <= '0;
        end else begin
            internal_reg <= i_data;
        end
    end

    // Combinational logic
    always_comb begin
        o_result = internal_reg;
        o_valid  = |internal_reg;
    end

endmodule
```

---

## Architecture Details

### Pipeline Stages (5-stage)

1. **IF (Instruction Fetch)**: PC management, instruction buffer, compressed instruction handling
2. **ID (Instruction Decode)**: Instruction decoding, register file read, immediate generation
3. **EX (Execute)**: ALU operations, multiply/divide, branch calculation
4. **MEM (Memory)**: Data memory access, load/store operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kerimturak/level-v](https://github.com/kerimturak/level-v) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

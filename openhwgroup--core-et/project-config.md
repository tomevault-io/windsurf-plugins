---
trigger: always_on
description: Clean reimplementation of the CORE-ET architecture. FPGA-friendly AND tapeout-friendly. Open-source tooling first.
---

# SoC Project Guidelines

## What this is

Clean reimplementation of the CORE-ET architecture. FPGA-friendly AND tapeout-friendly. Open-source tooling first.
Ainekko owns the CORE-ET IP and is reimplementing only the Ainekko-owned CORE-ET blocks in modern, minimal SystemVerilog.

This project targets **both FPGA prototyping and ASIC tapeout**. All tapeout infrastructure (DFT, clock gating, RAM wrappers, ECC, BIST hooks, power domain awareness) must be present. The approach is to use clean technology abstraction so the same RTL works across simulation, FPGA, and ASIC.

## Copyright header

Every source file must start with:
```
// Copyright (c) 2026 Ainekko
// SPDX-License-Identifier: Apache-2.0
```

## Repository layout

```
hw/ip/<block>/rtl/       Synthesizable RTL (one module per file, filename = module name)
hw/ip/<block>/dv/        Block-level testbench and tests
hw/ip/<block>/data/      CSR definitions, config (if needed)
hw/ip/<block>/doc/       Block-level spec (if needed)
hw/ip/tech_generic/<module>/  Technology primitives: behavioral (simulation, default)
hw/ip/tech_ice40/<module>/    Technology primitives: Lattice iCE40 FPGA
hw/ip/tech_xilinx/<module>/   Technology primitives: Xilinx 7-series / Ultrascale+
hw/ip/tech_asic/rtl/     Technology primitives: ASIC foundry-specific (private, future)
hw/top/                  Top-level chip integration
fpga/<project>/          Projects — combine IPs, target multiple backends (see Projects section)
  rtl/                     Shared project RTL
  <head>/                  Per-backend head (ice40/, xilinx/, verilator/, etc.)
dv/common/               Shared DV utilities (sim_ctrl.h)
dv/rtlcosim/             RTL co-simulation (old vs new module comparison)
mk/                      Build infrastructure (verilator.mk, yosys.mk)
fpga/                    FPGA constraints and wrappers
syn/                     Synthesis scripts
sw/                      Firmware and software
vendor/                  Third-party IP
docs/                    Project-wide documentation
```

RTL and its testbench live together under each IP block. Do not create separate global RTL and TB trees.

## Adding a new IP block

1. Create `hw/ip/<name>/README.md` — describe the module, what it is, and how to use it (ports, parameters, integration notes)
2. Create `hw/ip/<name>/rtl/<name>.sv`
3. Create `hw/ip/<name>/dv/Makefile` — set `TB_TOP`, `RTL_SRCS`, `CC_SRCS`, include `$(REPO_ROOT)/mk/verilator.mk`
4. Create `hw/ip/<name>/dv/<name>_test.cc` using `sim_ctrl.h` from `dv/common/`
5. The top-level `make test` auto-discovers all IP blocks with a `dv/Makefile`

Every IP block **must** have a `README.md` at its root (`hw/ip/<name>/README.md`) that describes:
- What the module does
- Parameters and their meaning
- Port interface (inputs, outputs, protocol)
- Usage example or integration notes
- Any design constraints or assumptions
- Differences from the original CORE-ET module (if reimplementing one)

## Original bug tracking

When analysis uncovers a bug in the original CORE-ET repository, document it immediately in a `BUGS.md` file at the relevant IP root (for example `hw/ip/<block>/BUGS.md`). If the file already exists, append a new entry instead of replacing it.

Every such entry must:
- state clearly in the heading that it is an **original repository bug**
- identify the original module(s) and the reimplemented module(s) affected
- describe the symptom, root cause, reachability, and system impact in report style
- cite the relevant original file paths and signal names or code fragments precisely enough for an engineer to audit quickly
- state whether the current reimplementation intentionally preserves the original behavior or intentionally diverges

Do not silently "fix" original-design bugs while claiming strict equivalence. If a correctness fix is chosen over equivalence, document that decision in both `BUGS.md` and the affected module `README.md`.

## Technology abstraction

Primitives whose implementation differs between FPGA and ASIC are **technology primitives**. They have the same module name and ports across all targets, but different implementations live in technology-specific directories:

```
hw/ip/tech_generic/prim_clk_gate/rtl/prim_clk_gate.sv    # behavioral (simulation, default)
hw/ip/tech_ice40/prim_clk_gate/rtl/prim_clk_gate.sv      # iCE40: negedge FF + AND
hw/ip/tech_xilinx/prim_clk_gate/rtl/prim_clk_gate.sv     # Xilinx: negedge FF + AND
hw/ip/tech_asic/rtl/prim_clk_gate.sv       # future: foundry ICG cell
```

Each technology tree must also carry a root `README.md` that explains the
primitive set at the right level:

- `hw/ip/tech_generic/README.md` describes each technology primitive,
  what it does, and the contract it presents to functional RTL. Treat this
  as the primitive-family behavioral/specification README.
- `hw/ip/tech_<specific>/README.md` describes how that technology
  implements the same primitive contract on that specific platform
  (resource mapping, vendor cells, FPGA-safe substitutions, limitations,
  and differences from `tech_generic`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openhwgroup/core-et](https://github.com/openhwgroup/core-et) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->

---
trigger: always_on
description: **Maintainer:** Sharjeel Imtiaz | PhD Student | Tallinn University of Technology (TalTech)
---

# CLAUDE.md — RISC-V Microarchitecture Lab
## Master Context for Claude Code Sessions

**Maintainer:** Sharjeel Imtiaz | PhD Student | Tallinn University of Technology (TalTech)
**Contact:** sharjeel.imtiaz@taltech.ee | sharjeelimtiazprof@gmail.com
**Repository:** riscv-microarchitecture-lab

---

## Project Vision

Design a complete, production-quality 64-bit RISC-V processor from architectural specification to physical chip implementation.

The processor will be:
- RV64IM to start, with a roadmap to RV64GC (IMAFDC)
- Out-of-order execution, 5-stage pipeline
- All privilege levels (M/S/U) per the RISC-V specification
- Capable of booting a small Linux kernel
- Open for community extension: UART, AI accelerators, crypto cores, etc.
- Available in two variants: standard and logic-locked (secure)

This is also a structured learning project. The goal is for Sharjeel to reach the point where he can independently start any verification task (pyuvm/cocotb testbench, SVA properties, RTL module) without AI assistance, then use Claude to go faster and deeper.

---

## Weekly Documentation Workflow

At the end of every work week, Claude Code generates two files for that week:

**Three files are generated per week:**

**1. `weekNN.md`** — `docs/weekly_notebooks/weekNN.md`
- Theory, concepts, architecture explanations, lessons learned, interview Q&A
- No command dumps — just learning content
- Public on GitHub, useful for other engineers and students

**2. `weekNN_commands.md`** — `docs/weekly_notebooks/weekNN_commands.md`
- Commands only, organized by task
- Every command has a one-line comment explaining what it does and why
- Meant to be a fast copy-paste reference during future sessions
- No theory, no prose — purely operational

**3. `weekNN_research_notebook.docx`** — `docs/weekly_notebooks/weekNN_research_notebook.docx`
- Everything woven together as one flowing readable document
- Commands appear inline where they are needed, not in a separate section
- Rich formatting: color tables, dark code blocks, Q&A boxes, page numbers
- Best for studying and interview prep

**How to trigger this in Claude Code:**

At the end of a session, say:
> "Generate the Week NN docs"

Claude Code generates all three files and places them in `docs/weekly_notebooks/`.

**What goes in each file:**

| Content | weekNN.md | weekNN_commands.md | .docx |
|---------|-----------|-------------------|-------|
| Objectives | Yes | No | Yes |
| Theory and explanations | Yes | No | Yes |
| Commands with context | No | Yes | Yes, inline |
| Progress tracker | Yes | No | Yes |
| Lessons learned | Yes | No | Yes |
| Interview Q&A | Yes | No | Yes |
| Next week plan | Yes | No | Yes |

**Naming convention:**
```
docs/weekly_notebooks/week01.md
docs/weekly_notebooks/week01_commands.md
docs/weekly_notebooks/week01_research_notebook.docx
```

---

## Phase Roadmap

```
Phase 0  [DONE]     Single-cycle RV32I  -  RTL complete, smoke test passing
Phase 1  [ACTIVE]   Single-cycle verification  -  pyuvm + cocotb + SVA + JasperGold + Genus + GLS + Conformal + Innovus
Phase 2  [NEXT]     RV64IM 5-stage pipeline  -  fetch/decode/execute/mem/writeback
Phase 3             Hazard unit  -  data hazards, forwarding, stall control
Phase 4             Branch prediction  -  BTB, static predictor first
Phase 5             Out-of-order  -  ROB, reservation stations, register renaming
Phase 6             Privilege levels  -  M/S/U, CSR file, trap/interrupt handling
Phase 7             Linux capable  -  MMU, SV39 virtual memory, timer
Phase 8             Security variant  -  logic locking, secure configuration mode
Phase 9             Peripherals  -  UART baseline, open peripheral interface
Phase 10            Physical  -  Genus synthesis, Innovus P&R, timing closure
```

**Phase 1 detailed flow (single-cycle as the learning vehicle):**
```
RTL simulation (Xcelium + smoke TB)
        |
        v
pyuvm functional verification (cocotb + scoreboard + coverage)
        |
        v
SVA assertion simulation (basic_props_checker bound into TB)
        |
        v
JasperGold formal proof on RTL (prove SVA properties exhaustively)
        |
        v
Genus synthesis (generate gate-level netlist + SDF delay file)
        |
        v
Cadence Conformal equivalence check (prove RTL == netlist, no logic changed)
        |
        v
Gate Level Simulation / GLS (simulate netlist with back-annotated SDF delays)
        |  - same pyuvm testbench, same smoke test
        |  - re-bind assertion checker on gate-level netlist
        |  - check: do properties still hold with real timing?
        v
JasperGold formal on gate-level netlist (prove properties on synthesized logic)
        |
        v
Innovus place and route (final physical implementation)
```

---

## Current State (as of Week 02)

**Completed:**
- Single-cycle RV32I RTL: pc, regfile, alu, alu_ctrl, immgen, inst_memory, data_memory, single_cycle_top
- Smoke testbench: `single_cycle_smoke_tb.sv`
- Smoke test passed: x1=5, x2=7, x3=12 — SMOKE PASS
- pyuvm skeleton: driver, monitor, scoreboard, sequencer, env, sequence, test
- SVA basics file: `basic_props.sv` — x0_immutable, opcode_nonzero_after_fetch
- VCD generated and verified in QuestaSim

**Known open issue:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sharjeelimtiaz27/riscv-microarchitecture-lab](https://github.com/Sharjeelimtiaz27/riscv-microarchitecture-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

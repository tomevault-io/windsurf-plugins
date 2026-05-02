---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commit conventions

**Do not add `Co-Authored-By:` trailers for AI agents** (Claude, Copilot, etc.)
when creating commits. The human author of record takes full ownership of the
change, and AI co-author trailers break the CLA Assistant (which requires every
listed author to individually sign the CLA — `noreply@anthropic.com` can't).
Just write a normal commit message with no AI attribution trailer.

## Project Overview

`arch-com` is a compiler for **ARCH**, a purpose-built hardware description language (HDL) for micro-architecture work. The compiler ingests `.arch` source files and emits deterministic, readable SystemVerilog. The language is explicitly designed to be generated correctly by LLMs from natural-language hardware descriptions.

Full specification: `doc/ARCH_HDL_Specification.docx`
Compact AI reference: `doc/Arch_AI_Reference_Card.docx`

---

## Target CLI (from spec)

The compiler binary is `arch`. MVP commands:

```
arch check F.arch          # type-check only (no output)
arch sim Tb.arch           # simulate (single core)
arch sim --parallel Tb.arch
arch sim --tlm-lt          # max speed, no timing
arch sim --tlm-at          # ns-accurate AT timing
arch sim --tlm-rtl         # full signal fidelity
arch sim --wave out.fst    # emit waveform (GTKWave/Surfer)
arch build F.arch          # emit SystemVerilog
arch formal F.arch         # emit SMT-LIB2
```

---

## Debugging Simulation Failures

**IMPORTANT: Use `--debug` instead of manually adding printf/`$display` to testbenches.**

When a simulation test fails or produces unexpected results, use the built-in debug instrumentation:

```bash
# Print all I/O port changes (top module only)
arch sim --debug Module.arch --tb tb.cpp

# Also print sub-module port changes (2 levels deep)
arch sim --debug --depth 2 Module.arch Sub.arch --tb tb.cpp

# Also print FSM state transitions with triggering conditions
arch sim --debug+fsm Module.arch --tb tb.cpp

# Combine: ports + FSM + 2 levels
arch sim --debug+fsm --depth 2 Module.arch Sub.arch --tb tb.cpp
```

The debug output covers:
- **Scalar ports**: `[cycle][Mod.port](in/out) 0x0 -> 0x1`
- **Vec ports**: `[cycle][Mod.data[2]](out) 0x0 -> 0xff` (per-element)
- **Bus ports**: `[cycle][Mod.s_axil_aw_valid](in) 0x0 -> 0x1` (with correct direction)
- **Wide ports (>64b)**: hex dump of all 32-bit words
- **FSM transitions**: `[FSM][Mod] IDLE -> RUN (start && ready)` (with condition)
- **Reset events**: `[cycle][Mod.rst](in) 0x0 -> 0x1`

Do NOT add `printf` or `$display` to C++ testbenches for debugging — `--debug` provides the same information automatically with zero code changes. Only add manual prints for test-specific protocol checking (e.g., verifying handshake sequences).

### Running Python/cocotb-style testbenches (`--pybind --test`)

`arch sim --pybind --test test_foo.py Foo.arch` compiles the ARCH design through pybind11 and runs a cocotb-compatible Python TB against it, with no Verilator/iverilog/VPI in the loop. A `cocotb_shim/cocotb/` package is on `PYTHONPATH` so plain `import cocotb` works. Supported surface: `@cocotb.test()`, `cocotb.start_soon`, `cocotb.start`, `cocotb.utils.get_sim_time`, and triggers `RisingEdge` / `FallingEdge` / `Timer` / `ClockCycles` / `Clock`. Signals expose `.value` that read/write as integers via `ArchSignalValue`.

Key behavioral deltas from real cocotb — scheduler is 1-tick-at-a-time (default 1 ns/tick), writes take effect immediately on the next `eval()` (no NBA region), logic is 2-state (no `X`/`Z`), edge detection is sampled per tick. See **[`doc/arch_sim_cocotb.md`](doc/arch_sim_cocotb.md)** for the complete API, portability rules, and a troubleshooting guide.

### Catching X-propagation from undriven inputs (`--inputs-start-uninit`)

When porting an SV design to ARCH, it's easy for a testbench to forget to drive a port. Under 4-state sim (Verilator/iverilog) this shows up as X-propagation; ARCH's native sim is 2-state so the bug is invisible unless you opt in:

```bash
arch sim --inputs-start-uninit Module.arch --tb tb.cpp
```

This implies `--check-uninit`. Every scalar input port starts in an "uninitialized" state; a TB opts each port in by calling the generated setter:

```cpp
dut.set_port_name(value);   // marks the port as driven
dut.port_name = value;      // does NOT mark it — will warn on read
```

If the design reads an input that the TB never drove (in a comb block, let binding, seq block, or latch), you get:

```
WARNING: read of uninitialized input 'port_name' — TB never called set_port_name()
```

Clock and Reset input ports are excluded (they're driven by the test harness lifecycle, not by the setter API). Bus ports and Vec ports are also excluded in v1.

A sibling flag `--check-uninit-ram` does the same for RAM cells — per-cell valid bitmap, `init:` cells pre-marked, warns once per RAM on the first read of an address that was never written. ROMs are exempt (they require `init:` at compile time). Both `--inputs-start-uninit` and `--check-uninit-ram` imply `--check-uninit`.

### Runtime bounds checking (always on)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arch-hdl-lang/arch-com](https://github.com/arch-hdl-lang/arch-com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

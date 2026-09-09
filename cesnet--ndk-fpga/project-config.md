---
trigger: always_on
description: Signpost for AI agents working in this repository. This file does not duplicate
---

# AGENTS.md

Signpost for AI agents working in this repository. This file does not duplicate
documentation — it points to where the real content lives, so check here before
searching blind.

## What this repository is

NDK-FPGA (CESNET's Network Development Kit for FPGA): a VHDL/SystemVerilog IP library
plus a Tcl/Make build system, used to build FPGA firmware for network acceleration
cards (`cards/`), assembled from reusable components (`comp/`) and a shared core
pipeline (`core/`). The repo also ships a runnable reference design, the "Minimal"
application (`apps/minimal`).

Verification is a mix of two styles: UVM/SystemVerilog (QuestaSim, `comp/**/ver/` and
`comp/**/uvm/`) and `cocotb` (Python-driven, `comp/**/cocotb/`) — both widespread, and
cocotb is what this repo's AI-agent skills currently cover.

Some IP (e.g. DMA Medusa) is closed-source and only available to CESNET and its partners — its
integration points live under `extra/`.

## Build system architecture

Read `build/readme.rst` for the full picture; the essentials:

- **`Modules.tcl`** files (one per component directory) declare that component's
  structure: `MOD` (own source files, in dependency order), `COMPONENTS`
  (subcomponents, as `[ENTITY ENTITY_BASE ARCHGRP]` triples), `PACKAGES` (VHDL
  packages, compiled first). `ARCHGRP` lets one `Modules.tcl` describe multiple
  architecture variants (commonly `FULL` vs `EMPTY`).
- Every buildable target has a plain `Makefile` setting `TOP_LEVEL_ENT` (and sometimes
  `TARGET`) that includes the shared `build/Makefile`, which dispatches to
  `Makefile.Vivado.inc` / `Makefile.Quartus.inc` / `Makefile.Synplify.inc`, or to
  `build/cocotb.mk` when `TARGET=cocotb`.
- `env.sh` (source from repo root) sets up the env vars local Python packages
  (`python/ofm`, `python/cocotbext`) need to resolve for PDM/pip installs. Only source
  it when creating a virtualenv — not when just running cocotb tests.

## Simulating/verifying VHDL — pick the right level first

There are two distinct kinds of cocotb simulation here. They share the same build
machinery but differ significantly in what `dut` is and how you configure them —
picking the wrong doc/skill for the task wastes time.

- **Single VHDL component** (a `comp/**/cocotb/` directory — a pipe, FIFO, MFB/MVB/
  AXI4-Stream block, etc.): `dut` in the test *is* the component under test. No card,
  no `NFBDevice`, no `CARD`/`DMA_TYPE`/`PCIE_CONF`.
  - **Read first**: `.agents/skills/ndk-cocotb-ver/SKILL.md` — structure conventions,
    generic randomized test pattern, backpressure/rate-limiter helpers, MI/Device
    Tree integration, reference implementations.
  - Docs: `doc/source/basic_cocotb_test.rst` (getting started), `doc/source/cocotbext.rst`
    (driver/monitor/utility library reference).

- **Whole-firmware / top-level simulation** (`apps/minimal/tests/cocotb/`): `dut` is
  the *entire card's* top-level entity. Tests go through `NFBDevice` and the MI/PCIe/
  Ethernet interfaces; the card and DMA engine variant are selected with `CARD`,
  `DMA_TYPE`, `PCIE_CONF` (not every combination is valid for every card — check the
  target card's `cards/<vendor>/<card>/config/card_conf.tcl` and
  `apps/minimal/tests/cocotb/top-level-sim.jenkinsfile`'s `EXTRA_COMBINATIONS` for
  which combinations actually exist and are CI-covered).
  - Docs: `doc/source/top_level_simulation.rst`.

- **Tips shared by both**: `doc/source/cocotb_tips_and_tricks.rst` — running a single
  test fast (`COCOTB_TEST_FILTER`), debug logging, random seed control, optional
  MFB/MVB/AXI4-Stream signals, throughput probes.
  - `TARGET=cocotb` (Questa/`vsim`) is the default simulator for a plain `make` on
    both kinds of test; NVC is opt-in via `make TARGET=nvc-sim` (or `make nvc-sim` for
    a component), not the default.

- Bus protocol references: `doc/source/mi.rst`, `doc/source/mfb.rst`,
  `doc/source/mvb.rst`, `doc/source/axi.rst`.
- **Alternative to cocotb**: some components use UVM/SystemVerilog instead
  (`comp/**/ver/`, `comp/**/uvm/`; `.fdo` scripts, `tbench/test_pkg.sv`,
  `ver_settings.py`; run via `vsim -do top_level.fdo`, orchestrated by Jenkins jobs
  in `tests/jenkins/ver_*.jenkins`). No agent skill covers this style yet — read the
  component's existing `ver`/`uvm` directory for conventions before touching it.

## Debugging without a waveform GUI

The `.rst` docs above point at the idiomatic *human* debugging path (`cocotb_test_sig.fdo`
+ a waveform viewer). You don't have a display, so use that path's agent-side
substitute instead — reading internal hierarchy signals directly for a few clock
cycles and logging them — documented in
`.agents/skills/ndk-cocotb-ver/SKILL.md` (section "Probing Internal DUT Signals (No
Waveform GUI)"), which covers both the component-level and top-level-sim variants.

## Before considering a change done

- Changed Python (including cocotb tests)? Run `tests/ci/pycodestyle.sh` (flake8 +
  mypy against `python/ofm`) — CI enforces this.
- Changed VHDL? Check it with `vhdl_ls` first (`.agents/skills/vhdl-lint/SKILL.md`),
  then see "Per-component simulation/synthesis check" below for a deeper check. CI
  also enforces style with `vsg` (`tests/ci/vsg_config.yaml`); if unavailable locally,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CESNET/ndk-fpga](https://github.com/CESNET/ndk-fpga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

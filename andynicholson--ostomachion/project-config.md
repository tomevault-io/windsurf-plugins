---
trigger: always_on
description: Guidance for AI assistants and contributors working on Ostomachion from a
---

# CLAUDE.md — Ostomachion FPGA Platform Guide

Guidance for AI assistants and contributors working on Ostomachion from a
**senior FPGA architect** perspective.  For setup walkthroughs see
[GETTING_STARTED.md](GETTING_STARTED.md); for API and tree detail see
[DEVELOPER.md](DEVELOPER.md).

---

## Project identity

Ostomachion is a **NEORV32 RISC-V SoC + Zephyr RTOS** platform on the
**Opal Kelly XEM7310-A200** (Xilinx Artix-7 XC7A200T), with a DMA-driven
**4096-point xFFT** accelerator pipeline in fabric.

Design philosophy: a small set of composable, interlocking parts — fourteen
architectural layers documented in [README.md](README.md#the-fourteen-pieces).

- **License:** GPL-3.0-or-later or commercial ([LICENSE](LICENSE))
- **NEORV32:** upstream submodule at `neorv32/` — do not edit casually
- **Zephyr:** pinned via [west.yml](west.yml)

---

## Non-negotiable architecture rules

1. **No hand-edited Vivado checkpoints in the tree.**  The block design is
   recreated from Tcl on every build.  There must be no saved `.bd` files,
   `.xpr` projects, or GUI-generated wrappers under version control.

2. **Block design source of truth:** [`fpga/xem7310/ostomachion_bd.tcl`](fpga/xem7310/ostomachion_bd.tcl).
   All Xilinx IP (SmartConnect, AXI DMA, xfft, BRAM controllers, INTC, GPIO,
   clock wizard) lives inside the generated `ostomachion_bd_wrapper`.

3. **Board RTL source of truth:** [`fpga/xem7310/xem7310_top.vhd`](fpga/xem7310/xem7310_top.vhd).
   Hand-written VHDL for clocks, pads, NEORV32, bridge, FrontPanel, and UART/pipe
   bridges.  Do not put application logic inside the BD canvas.

4. **Simulation wrapper is separate:** [`rtl/neorv32_wrapper.vhd`](rtl/neorv32_wrapper.vhd)
   wraps `neorv32_top` for GHDL — it is not the FPGA top.

5. **Batch build entry point:** [`fpga/xem7310/build.tcl`](fpga/xem7310/build.tcl)
   invoked by `make fpga-synth`.  Quality gates in
   [`fpga/xem7310/check_build.tcl`](fpga/xem7310/check_build.tcl).

6. **All development happens on branches, merged via squash PRs.**  `master`
   (and `main`/`develop`) are protected: never commit or push directly to them.
   Every change — code, RTL, docs — lands through this flow:

   ```bash
   git checkout -b <type>/<short-topic>     # feat/, fix/, chore/, docs/, ci/
   # ... edit, build, verify ...
   git push -u origin <branch>
   gh pr create --base master --fill        # open the PR
   gh pr merge   --squash --delete-branch    # squash-merge once CI + review pass
   ```

   - **Squash-merge only** — one logical change becomes one commit on `master`,
     so history stays linear and bisectable.  Do not use merge commits or
     rebase-merge.
   - The PR must pass the automatic CI jobs (below) before merge.  Self-hosted
     Vivado synthesis is manual; run it for any RTL/constraints/BD change and
     cite the WNS/WHS/DRC result in the PR.
   - Branch names are typed: `feat/`, `fix/`, `chore/`, `docs/`, `ci/`,
     `refactor/`.  Delete the branch on merge (`--delete-branch`).
   - Record any non-obvious architectural decision or revert as a clause in
     [REVIEW.md](REVIEW.md) (the standing architectural contract / open ledger)
     in the same PR — treat docs as code.

---

## Fabric hierarchy

```
xem7310_top.vhd
├── IBUFDS (200 MHz LVDS oscillator)
├── neorv32_top + xbus2axi4_bridge → s_axi_cpu
├── ostomachion_bd_wrapper (Vivado-generated from ostomachion_bd.tcl)
│   ├── clk_wiz_0 (MMCM → 100 MHz aclk)
│   ├── proc_sys_reset_0
│   ├── axi_smc (SmartConnect: 3 masters × 6 slaves)
│   ├── axi_dma_0 (MM2S + S2MM; S2MM source mux'd in top RTL)
│   ├── xfft_0 (4096-pt forward) → [filter, in top RTL] → xfft_1 (inverse)
│   ├── tx_bram / rx_bram / coeff_bram (true-dual-port) + BRAM controllers
│   ├── axi_intc → mext_irq_o
│   └── axi_gpio (ch1: xfft aresetn gate + filter bypass; ch2: overflow in)
├── spectral_filter + cmpy_normalizer (per-bin H[k]·X[k], Q2.30→Q1.15) [top RTL]
├── bypass mux (selects S2MM source: xfft_0 bins vs xfft_1 filtered IFFT) [top RTL]
├── okHost / okWire* / okPipe* (FrontPanel)
├── fp_uart_bridge (NEORV32 UART ↔ FrontPanel pipes)
├── fp_fft_pipe_bridge (host FFT pipe I/O)
├── fft_beat_counter ×2 (xfft_0 → WireOut 0x22, xfft_1 → WireOut 0x27)
└── overflow sticky latch ×2 (xfft_0 → GPIO2 bit0, aggregated → bit1)
```

**Boundary discipline:** CPU sees the accelerator only through the AXI map
(DMA registers + BRAM windows + INTC).  Host PC sees FrontPanel wires/pipes
in parallel — never substitute host pipes for CPU DMA paths without explicit
RTL design.

---

## Clock and reset domains

| Domain | Source | Consumers |
|--------|--------|-----------|
| `sys_clk` / `aclk` | 200 MHz LVDS → MMCM → 100 MHz | AXI, DMA, xfft, BRAM |
| NEORV32 core clock | Same 100 MHz from BD `clk_o` | CPU, peripherals |
| `okClk` | FrontPanel USB clock | WireIn/WireOut sampling |
| Async FIFO CDC | `fp_uart_bridge`, `fp_fft_pipe_bridge` | UART/pipe ↔ NEORV32 |

**Reset chain:** external reset → `proc_sys_reset_0` → peripheral `aresetn`.
The xfft core has an additional **software-gated** reset via AXI GPIO bit 0
(see [ACCEL_ARCH.md](ACCEL_ARCH.md)).

**CDC rule:** do not sample AXI-side beat counters directly into FrontPanel
without registered staging — `fft_beat_counter` outputs are registered on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andynicholson/Ostomachion](https://github.com/andynicholson/Ostomachion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project boots nommu Linux (kernel 6.6.83) on a NEORV32 RV32IMAC soft-core FPGA — the first known Linux boot on NEORV32. The NEORV32 has no MMU and no S-mode. Getting Linux running required 16 kernel patches across arch/riscv, scheduler, RCU, init, and drivers. We also found and fixed a [SC.W return value bug](https://github.com/stnolting/neorv32/pull/1520) in NEORV32's bus reservation station, enabling native atomic instructions in the kernel; the fix is now merged upstream and included in v1.12.9.

The `neorv32` submodule is at the post-`v1.12.9` `origin/main` HEAD (currently `e0739e63`) plus three local RTL patches in `neorv32_patches/` (`0001`, `0002`, `0004`). See `neorv32_patches/README.md` for the full inventory and rationale. The optional diagnostic counter patch (`0003`) is also included by default — it can be dropped without affecting boot.

**D-cache is disabled** (`DCACHE_EN => false` in `rtl/ax301_top.vhd`). The new write-back D-cache architecture (PR #1513) requires burst-capable memory to be a net win; against the simple non-burst SDRAM controller in this project it is a net loss and triggers a `ktime_get_coarse_real_ts64` seqcount-retry livelock on hot kernel paths. With D-cache off the kernel boots cleanly to `nommu#` in ~36 s wall time (3× faster than the v1.12.9 baseline of ~118 s with the older write-through D-cache enabled).

Target hardware: Heijin AX301 board with Altera Cyclone IV EP4CE6 FPGA, 32 MB SDRAM, 50 MHz.

## Hardware

**Board:** AX301 (Cyclone IV EP4CE6F17C8)
**Programmer:** USB-Blaster (`09fb:6001`), attached to WSL2 via `usbipd`
**UART:** PL2303 at `/dev/ttyUSB0`
**Key peripherals:** 32 MB SDRAM (HY57V2562GTR), SPI Flash (M25P16)

## Repository Structure

```
see_neorv32_run_linux/
├── tools/openFPGALoader/    — openFPGALoader source (build from source)
├── neorv32/                 — NEORV32 RTL source (git submodule → stnolting/neorv32, post-v1.12.9 origin/main)
├── linux-6.6.83.tar.xz     — Linux kernel tarball
├── rtl/                     — Custom RTL (ax301_top.vhd, sdram_ctrl.v, wb_sdram_ctrl.v)
├── quartus/                 — Quartus project (neorv32_demo.qsf/qpf/sdc)
├── kernel/                  — neorv32_nommu.patch (16 kernel patches)
├── board/                   — DTS, defconfig, UART driver, inject_driver.sh
├── sw/stage2_loader/        — Stage2 xmodem loader (C, must fit 8 KB)
├── sw/initramfs/            — Minimal init (C, builds neo_initramfs.cpio.gz)
├── host/                    — boot_linux.py, test_shell.py
└── output/                  — Build outputs go here (initially empty)
```

## Complete Build-from-Source Flow

All source code is included. Build order matters — later steps depend on earlier outputs.

**Submodule:** `neorv32/` is a git submodule pointing to `stnolting/neorv32` at a post-`v1.12.9` `origin/main` commit (`e0739e63`). After cloning this repo, run `git submodule update --init --recursive` and then apply the patches in `neorv32_patches/` before building:

```bash
git submodule update --init --recursive
cd neorv32
for p in ../neorv32_patches/*.patch; do git apply "$p"; done
cd ..
```

### Prerequisites

- Intel Quartus Prime Lite 21.1+ (`~/intelFPGA_lite/21.1/quartus/bin` in PATH)
- xPack RISC-V GCC 14.2.0 (`riscv-none-elf-gcc`) — for kernel and stage2. **MUST use this specific version** — see "Compiler constraint" below. Install from https://github.com/xpack-dev-tools/riscv-none-elf-gcc-xpack and put the `bin/` directory on `$PATH`.
- A glibc/musl RISC-V Linux toolchain (`riscv32-buildroot-linux-gnu-gcc` or similar) — for initramfs init ONLY (needs static-PIE support; the bare-metal `riscv-none-elf-` toolchain cannot produce PIE executables). Buildroot is the easiest source; its `output/host/bin/` prefix works as-is. Override with `make RISCV_LINUX_PREFIX=...` when building `sw/initramfs/`.
- CMake, libftdi1-dev, libusb-1.0-0-dev (for openFPGALoader)
- Device tree compiler: `dtc`
- Python 3 with `pyserial`

**Two toolchains are required:** The bare-metal `riscv-none-elf-` toolchain cannot produce PIE executables. The initramfs `/init` is a Linux userspace binary that must be built as static-PIE with the Buildroot Linux toolchain. Do NOT substitute one for the other.

**Compiler constraint (critical):** The kernel MUST be built with xPack `riscv-none-elf-gcc` 14.2.0. Building with Buildroot's `riscv32-buildroot-linux-gnu-gcc` 12.4.0 produces a kernel that hangs in `free_initmem()` — identical source, patches, and .config, but GCC 12.4.0 generates machine code that deadlocks on NEORV32. Symptoms: debug marker `L` prints (system_state = RUNNING), but `M` (after free_initmem) never appears; the Buildroot-built kernel also runs ~2x slower and triggers `sched: RT throttling activated`.

### Step 1: Build openFPGALoader

The system-installed `openfpgaloader` (v0.12.0) does **NOT** recognise EP4CE6. Must build from source.

```bash
cd tools/openFPGALoader
mkdir build && cd build
cmake ..
make -j$(nproc)
# Binary: tools/openFPGALoader/build/openFPGALoader
```

### Step 2: Build FPGA bitstream


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [14sea/see_neorv32_run_linux](https://github.com/14sea/see_neorv32_run_linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->

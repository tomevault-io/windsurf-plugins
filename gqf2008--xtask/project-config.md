---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`xtask` is a `#![no_std]` preemptive multitasking RTOS-style kernel for embedded targets, modeled on FreeRTOS. It compiles together with the application. Single physical hardware thread: priority + time-slice scheduling, high-priority preemption, fair round-robin among equal priorities. The design/rationale (including the assembly context-switch code) is documented in `Xtask.md` — read it before touching scheduling or porting code.

Rust **nightly** is required (`rust-toolchain` pins `nightly`; `lib.rs` uses many `#![feature(...)]` gates). Pure-logic unit tests (semaphore, bus, delay queue, software timers) run on the host via a `cfg(test)` `HostPorting` mock: `cargo test --lib --target x86_64-pc-windows-msvc` (any host triple works; do not omit `--lib`, examples only build for embedded targets). Everything else is verified by building and flashing examples to real boards.

## Build & run

Everything is selected with a **chip feature** + a **target triple**. You must pass both; the kernel does not build without a chip feature. `build.rs` copies the matching `src/chip/<chip>/memory.x` linker script into `OUT_DIR`.

```bash
# General form
cargo run --example <EXAMPLE> --features <CHIP> --target <TRIPLE> --release

# gd32vf103 (longan-nano)
cargo run --example led --features gd32vf103 --target riscv32imac-unknown-none-elf --release
# stm32f4 (greenpill; note multitask.rs itself is a gd32vf103-only example)
cargo run --example multitask_greenpill --features stm32f4 --target thumbv7em-none-eabihf --release
# stm32f1 (bluepill)
cargo run --example multitask_bluepill --features stm32f1 --target thumbv7m-none-eabi --release
# qemu_riscv (QEMU virt machine — the only port that EXECUTES, not just links;
# run under QEMU and self-exits via SiFive test device):
qemu-system-riscv32 -M virt -nographic -bios none -kernel     target/riscv32imac-unknown-none-elf/release/examples/qemu_pingpong
# rp2040 (revived 2026-08-23 on rp2040-hal 0.9): the old 0.5.0 pin depended on a yanked
# critical-section 0.2.x; 0.9+ uses 1.x. The board BSP is the repo's own bsp_pins! macro.
cargo run --example multitask_rp_pico --features rp2040 --target thumbv6m-none-eabi --release
```

`--release` is normal for flashing (both profiles use `opt-level = "z"`, `lto = true`). Some examples need extra features, e.g. the software-timer example: `--features gd32vf103,timer`.

A default target (`thumbv7em-none-eabihf`) and per-target `runner` (probe-run / gdb+openocd / elf2uf2-rs) are set in `.cargo/config.toml`; `cargo run` uses the runner to flash. OpenOCD configs and GDB scripts live in `debug/<chip>/`. Chip HALs are mostly crates, but `gd32vf103xx-hal` is a local path dep at `hal2/gd32vf103xx-hal`.

Chip features: `gd32vf103`, `stm32f1`, `stm32f4`, `stm32h7`, `cm32m4`, `rp2040`, `ch32v103`/`ch32v203`/`ch32v307`, `ch583` (CH58x / QingKe V4A; build-verified 2026-09-19, ROM boot header checked by `ci/check_wch_boot.py`), `esp32c3` (RV32IMC — build with `riscv32imc`: the `riscv32imac` artifact carries A-extension instructions the chip can't execute), plus `qemu_riscv` (QEMU virt; **execution-verified** — `ci/gate.sh` runs it; others build-verified 2026-08-23; real-board verification pending for f4/f1 constants, h7 timeline, cm32m4/rp2040). Non-chip features: `timer` (software timers), `tlsf` (global allocator backend swap to the hand-written mini TLSF, ch28), `debug_task`, `fs` (fatfs), `net` (smoltcp), `usb`, `ble`, `rtt_log` / `stdout_log`, board BSPs (`longan_nano`, `bluepill`, `greenpill`, `rp_pico`).

## Architecture

Layered, bottom-up:

- **`src/port.rs` — the portability seam.** Trait `Portable` defines the ~10 primitives the kernel needs (memory barrier, critical section `free`, enable/disable interrupt, `start_scheduler`, software `irq`, `systick`, `delay_us`, `save_context`; plus six defaulted tickless methods — `tickless_supported`/`tickless_arm_delta`/`tickless_stop_timer`/`tickless_wait`, the leave-idle boundary hook `tickless_leave_idle` (measured catch-up + periodic restore, called by `do_schedule` when leaving idle), and `tickless_resume_periodic` (self-checking re-enable before the fallback spin), all default to constant-tick no-ops, so the 14 ports compile unchanged, ch29). The type alias `Porting` is bound to exactly one chip implementation via `#[cfg(feature = ...)]`. **All kernel code calls `Porting::...` — never a chip directly.** To port to a new chip: implement `Portable` + provide `memory.x` under `src/chip/<chip>/`, then add the feature wiring in `port.rs`, `chip/mod.rs`, `build.rs`, and `Cargo.toml`. For a **RISC-V target without the A extension** (CH572/青稞无 A 档位、esp32c3 真身) use target `riscv32imc-unknown-none-elf`: the kernel's atomic RMW already goes through `atomic-polyfill`, whose fallback needs a `critical-section` impl — that lives in `src/arch/riscv/critical.rs` (`mstatus.MIE` save/restore, single-core semantics), is compiled for **every riscv32 target** (`AtomicU64`/`AtomicI64` go through the polyfill even on riscv32imac), and `ci/gate.sh` builds that target so it can't silently rot.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gqf2008/Xtask](https://github.com/gqf2008/Xtask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

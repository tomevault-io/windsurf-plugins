---
trigger: always_on
description: fstart is a next-generation firmware framework in Rust. Board `.ron` files are the
---

# AGENTS.md — fstart firmware framework

## Project Overview

fstart is a next-generation firmware framework in Rust. Board `.ron` files are the
single source of truth — the build system reads them and generates stage entry points,
driver instantiation, and linker scripts. No hand-written stage code.

For domain inspiration, reference codebases are available at `~/src/coreboot` (C,
payload/stage architecture) and `~/src/u-boot` (C, device-tree-driven board defs).

## Design Documents

- **[Driver Model](docs/driver-model.md)** — typed device/driver architecture inspired
  by coreboot's device tree and U-Boot's uclass/ops model, redesigned for Rust's type
  system. Covers the `Device` trait, associated `Config` types, codegen-produced
  `Devices`/`StageContext` structs, bus hierarchies, and Rigid vs Flexible dispatch.
- **[Continuation Plan](docs/continuation-plan.md)** — what has been built, what
  remains, and the recommended order of work. Includes phase-by-phase status and
  detailed next-step descriptions.

## Environment

This is a NixOS system. Tools not on `$PATH` (e.g., `qemu`, `file`, `objdump`) must
be run via `nix-shell`:

```bash
nix-shell -p qemu file --run "qemu-system-riscv64 -M virt -bios firmware.bin"
nix-shell -p binutils --run "objdump -d target/.../fstart-stage"
```

## Build / Run / Check Commands

```bash
# Check all host-side crates (fast, no cross-compile env needed)
cargo check --workspace --exclude fstart-stage \
    --exclude fstart-platform-riscv64 --exclude fstart-platform-aarch64 \
    --exclude fstart-platform-armv7 --exclude fstart-runtime

# Build a specific board (sets FSTART_BOARD_RON, cross-compiles with -Z build-std=core)
cargo xtask build --board qemu-riscv64
cargo xtask build --board qemu-aarch64
cargo xtask build --board qemu-armv7
cargo xtask build --board bananapi-m1
cargo xtask build --board qemu-riscv64 --release

# Build and launch in QEMU
cargo xtask run --board qemu-riscv64
cargo xtask run --board qemu-aarch64
cargo xtask run --board qemu-armv7

# Clippy — host crates only (fstart-stage and platform crates need cross-compile)
cargo clippy --workspace --exclude fstart-stage \
    --exclude fstart-platform-riscv64 --exclude fstart-platform-aarch64 \
    --exclude fstart-platform-armv7 --exclude fstart-runtime -- -D warnings

# Format
cargo fmt --all
cargo fmt --all -- --check   # CI-style check

# Run tests (47 codegen + 14 FFS; add more with #[cfg(test)])
cargo test --workspace --exclude fstart-stage --exclude fstart-runtime \
    --exclude fstart-alloc \
    --exclude fstart-platform-riscv64 --exclude fstart-platform-aarch64 \
    --exclude fstart-platform-armv7

# Run a single test by name
cargo test --package fstart-types -- test_name_here

# Run a single test file (integration test)
cargo test --package fstart-codegen --test integration_test_name
```

Note: `fstart-stage`, `fstart-runtime`, and platform crates are `no_std` `#![no_main]`
binaries — they cannot be tested with `cargo test` on the host. Test logic for these
via `fstart-types` or `fstart-codegen` (which are `std`-capable).

## Workspace Layout (22 crates)

### Host-side (std) crates

| Crate | Purpose |
|---|---|
| `xtask` | Build orchestrator, QEMU launcher, eGON patching |
| `fstart-codegen` | RON→Rust codegen, linker script gen (used in build.rs) |
| `fstart-device-registry` | Aggregates all driver `Config` types for codegen |

### Shared crates (std feature for host, no_std for target)

| Crate | Purpose |
|---|---|
| `fstart-types` | `BoardConfig`, `MemoryMap`, `StageLayout`, all shared types |
| `fstart-ffs` | Firmware filesystem reader/builder |

### Target (no_std) crates — core infrastructure

| Crate | Purpose |
|---|---|
| `fstart-stage` | Final binary — `include!`s generated code |
| `fstart-runtime` | `#[panic_handler]` |
| `fstart-services` | Trait defs: `Console`, `BlockDevice`, `Timer`, `Device`, `BusDevice` |
| `fstart-capabilities` | Capability impls (ConsoleInit, DramInit, PayloadLoad, etc.) |
| `fstart-arch` | Architecture utils: `udelay`, `sdelay`, `mdelay`, `halt` (feature-gated: `armv7`, `aarch64`, `riscv64`) |
| `fstart-log` | Logging macros (`info!`, `error!`, etc.) backed by ufmt |
| `fstart-mmio` | MMIO register access helpers |
| `fstart-crypto` | Signature verify, hashing |
| `fstart-alloc` | Allocator (skeleton) |

### Target (no_std) crates — platform / SoC

| Crate | Purpose |
|---|---|
| `fstart-platform-riscv64` | `_start` entry for RISC-V 64 |
| `fstart-platform-aarch64` | `_start` entry for AArch64 |
| `fstart-platform-armv7` | `_start` entry for ARMv7 (optional `sunxi` feature) |
| `fstart-soc-sunxi` | Allwinner eGON boot header, FEL support, boot media detection |

### Target (no_std) crates — individual drivers

| Crate | Driver | Services |
|---|---|---|
| `fstart-driver-ns16550` | NS16550(A) UART | `Console` |
| `fstart-driver-pl011` | ARM PL011 UART | `Console` |
| `fstart-driver-designware-i2c` | DesignWare APB I2C | `I2cBus` |
| `fstart-driver-sunxi-ccu` | Allwinner A20 CCU | `ClockController` |
| `fstart-driver-sunxi-a20-dramc` | Allwinner A20 DRAM controller | `MemoryController` |
| `fstart-driver-sunxi-mmc` | Allwinner A20 SD/MMC | `BlockDevice` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fstart-io/fstart](https://github.com/fstart-io/fstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->

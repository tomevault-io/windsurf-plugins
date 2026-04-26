---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Rule #1: NEVER chain pkill with other commands

**CRITICAL**: `pkill` MUST always be run as its own separate, standalone Bash command. NEVER combine with `&&`, `;`, `||`. When `pkill` finds no matching process it returns exit code 1, which prevents chained commands from executing.

```bash
# CORRECT: Two separate Bash calls
pkill -9 -f qemu-system    # Call 1 (may return exit code 1, that's OK)
sleep 2

qemu-system-x86_64 ...     # Call 2 (separate invocation)

# WRONG: Chained together
pkill -9 -f qemu-system; sleep 2; qemu-system-x86_64 ...
```

## VeridianOS Overview

Next-generation microkernel OS in Rust. Capability-based security, user-space drivers, multi-arch (x86_64, AArch64, RISC-V). All phases (0-12) complete, v0.25.1. See CLAUDE.local.md for current state.

## Essential Commands

### Building

```bash
# Recommended: build script
./build-kernel.sh all dev          # All architectures, dev mode
./build-kernel.sh x86_64 release   # Specific arch, release mode

# Manual
cargo build --target targets/x86_64-veridian.json -p veridian-kernel -Zbuild-std=core,compiler_builtins,alloc
cargo build --target aarch64-unknown-none -p veridian-kernel
cargo build --target riscv64gc-unknown-none-elf -p veridian-kernel
```

**Notes**: x86_64 requires custom target JSON with kernel code model (R_X86_64_32S relocation fix). Kernel linked at 0xFFFFFFFF80100000.

### Running in QEMU (VERIFIED -- QEMU 10.2)

#### x86_64 (UEFI boot -- requires OVMF + disk image)

x86_64 uses UEFI boot via bootloader 0.11+. **CANNOT** use `-kernel` flag directly.

```bash
# Build (creates UEFI disk image automatically)
./build-kernel.sh x86_64 dev

# Run (serial only, ALWAYS use -enable-kvm)
qemu-system-x86_64 -enable-kvm \
    -drive if=pflash,format=raw,readonly=on,file=/usr/share/edk2/x64/OVMF.4m.fd \
    -drive id=disk0,if=none,format=raw,file=target/x86_64-veridian/debug/veridian-uefi.img \
    -device ide-hd,drive=disk0 \
    -serial stdio -display none -m 256M

# With framebuffer (remove -display none for 1280x800 BGR UEFI GOP)
# With GDB: add -s -S (server :1234, start paused)
# With debug exit: add -device isa-debug-exit,iobase=0xf4,iosize=0x04
# With BlockFS rootfs (needs 2048M RAM):
#   add -drive file=target/rootfs-blockfs.img,if=none,id=vd0,format=raw -device virtio-blk-pci,drive=vd0
```

#### AArch64 (direct kernel boot)

```bash
./build-kernel.sh aarch64 dev
qemu-system-aarch64 -M virt -cpu cortex-a72 -m 256M \
    -kernel target/aarch64-unknown-none/debug/veridian-kernel \
    -serial stdio -display none
# Add -device ramfb for graphical display | Add -s -S for GDB
```

#### RISC-V 64 (OpenSBI + kernel)

```bash
./build-kernel.sh riscv64 dev
qemu-system-riscv64 -M virt -m 256M -bios default \
    -kernel target/riscv64gc-unknown-none-elf/debug/veridian-kernel \
    -serial stdio -display none
# Add -device ramfb for graphical display | Add -s -S for GDB
```

#### QEMU Quick Reference

| Arch | Boot | Firmware | Image | KVM |
|------|------|----------|-------|-----|
| x86_64 | UEFI disk | OVMF.4m.fd `-drive if=pflash` | `target/x86_64-veridian/debug/veridian-uefi.img` | `-enable-kvm` (REQUIRED) |
| AArch64 | Direct `-kernel` | None | `target/aarch64-unknown-none/debug/veridian-kernel` | N/A (TCG) |
| RISC-V | `-kernel` + `-bios default` | OpenSBI | `target/riscv64gc-unknown-none-elf/debug/veridian-kernel` | N/A (TCG) |

**Expected**: All 3 archs boot Stage 6 BOOTOK, 29/29 tests. x86_64 shows Ring 3 entry.

#### QEMU 10.2 Pitfalls
- **DO NOT** use `timeout` -- causes "drive exists" errors. Use background+kill: `cmd </dev/null > log 2>&1 &; PID=$!; sleep N; kill $PID`
- **DO NOT** use `-kernel` for x86_64 -- fails with "PVH ELF Note" error
- **DO NOT** use `-bios` instead of `-drive if=pflash` -- different semantics
- **DO NOT** use `-drive` without explicit ID -- conflicts with pflash
- **DO NOT** use `-cdrom` alongside `-drive` on same bus
- **DO NOT** use `cargo run` for x86_64 -- wrong runner
- **ALWAYS** `pkill -9 -f qemu-system; sleep 3` before re-running
- **ALWAYS** use `-enable-kvm` for x86_64 (TCG is ~100x slower)

**PS/2 keyboard**: Polling (ports 0x64/0x60). APIC replaces PIC so IRQ-based keyboard doesn't work. Input from both serial and keyboard.

### Testing

```bash
# Format and lint (always run these)
cargo fmt --all
cargo clippy --target x86_64-unknown-none -p veridian-kernel -- -D warnings
cargo clippy --target aarch64-unknown-none -p veridian-kernel -- -D warnings
cargo clippy --target riscv64gc-unknown-none-elf -p veridian-kernel -- -D warnings

# Host-target tests (4,095+ passing)
cargo test

# NOTE: Automated kernel tests blocked by Rust toolchain lang items limitation
# Manual kernel testing via QEMU commands above
```

### Development Tools

```bash
rustup toolchain install nightly-2025-01-15
rustup component add rust-src llvm-tools-preview
cargo install bootimage cargo-xbuild cargo-watch cargo-expand cargo-audit cargo-nextest
```

## Architecture

### Microkernel Design
- **Core**: Memory management, scheduling, IPC, hardware abstraction
- **User-space drivers**: Capability-controlled MMIO, interrupt forwarding, IOMMU DMA

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/doublegate) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

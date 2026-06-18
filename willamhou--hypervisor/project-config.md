---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ARM64 Type-1 bare-metal hypervisor in Rust (`no_std`) + ARM64 assembly. One codebase, two compile-time personalities:

- **NS-EL2 hypervisor** — boots Linux 6.12.12 to BusyBox (4 vCPUs, virtio-blk/net, multi-VM with per-VM Stage-2 + VMID TLBs + L2 vswitch), Android (PL031 RTC, Binder, 1GB RAM), and a FF-A v1.1 proxy (pKVM-compatible).
- **S-EL2 SPMC** (`sel2`) — runs as TF-A BL32, manages Secure Partitions at S-EL1 (SP1 Hello, SP2 IRQ, SP3 Relay), full FF-A v1.1: DIRECT_REQ (incl. SP↔SP with cycle detection), memory sharing (SHARE/LEND/DONATE/RETRIEVE/RELINQUISH/RECLAIM, NWd↔SP and SP↔SP), secure vIRQ injection, NS interrupt preemption.

End state target: TF-A BL31+SPMD @ EL3 → our SPMC @ S-EL2 → SPs @ S-EL1, alongside pKVM @ NS-EL2 → Linux/Android @ NS-EL1. Validated E2E against real pKVM (`ffa_test.ko`: 35/35 PASS). See Roadmap and `DEVELOPMENT_PLAN.md`.

## Build Commands

```bash
make              # Build hypervisor
make run          # Build + run in QEMU — runs 34 test suites automatically (exit: Ctrl+A then X)
make run-linux    # Boot Linux guest (--features linux_guest, 4 vCPUs on 1 pCPU, virtio-blk)
make run-linux-smp # Boot Linux guest (--features multi_pcpu, 4 vCPUs on 4 pCPUs)
make run-multi-vm # Boot 2 Linux VMs time-sliced (--features multi_vm)
make run-android  # Boot Android-configured kernel (PL031 RTC, Binder, minimal init, 1GB RAM)
make run-guest GUEST_ELF=/path/to/zephyr.elf  # Boot Zephyr guest (--features guest)
make debug        # Build + run with GDB server on port 1234
make check / clippy / fmt / clean

# Secure-world / TF-A chain (all Docker-based, TCG-only — KVM cannot virtualize EL3/Secure)
make build-qemu        # Build QEMU 9.2.3 from source (one-time)
make build-tfa-bl33    # TF-A flash.bin, PRELOADED_BL33_BASE=0x40200000
make run-tfa-linux     # TF-A → hypervisor (BL33) → Linux (needs build-tfa-bl33)
make build-tfa-spmc    # TF-A + real SPMC (BL32) + SP Hello/IRQ/Relay
make run-spmc          # TF-A → our SPMC (BL32) at S-EL2
make build-tfa-full    # TF-A real SPMC (BL32) + preloaded BL33 hypervisor
make run-tfa-linux-ffa # TF-A → SPMC → hypervisor (BL33) → Linux (FF-A discovery)
make build-spmc / build-sp-hello / build-sp-irq / build-sp-relay  # individual binaries

# pKVM integration (AOSP kernel as BL33)
make build-pkvm-kernel # AOSP android16-6.12 kernel for pKVM (Docker, ~15-30min first time)
make build-tfa-pkvm    # flash-pkvm.bin (ARM_LINUX_KERNEL_AS_BL33)
make run-pkvm          # pKVM (NS-EL2) + our SPMC (S-EL2), AOSP kernel as BL33
make run-pkvm-ffa-test # pKVM with FF-A test module (35/35 PASS)

# AVF / crosvm (needs ARM64 host + /dev/kvm for full validation)
make build-crosvm / build-crosvm-initramfs / run-crosvm
```

**Feature flags** (Cargo, selected by Makefile targets):
- `(default)` — unit tests only, no guest boot
- `guest` — Zephyr loading
- `linux_guest` — Linux guest: DynamicIdentityMapper, GICR trap-and-emulate, virtio-blk/net
- `multi_pcpu` (⊃ linux_guest) — 1:1 vCPU↔pCPU affinity, PSCI boot, TPIDR_EL2 context, SpinLock devices
- `multi_vm` (⊃ linux_guest) — 2 VMs time-sliced on 1 pCPU, per-VM Stage-2/VMID/DeviceManager
- `sel2` — S-EL2 SPMC mode: BL32, `boot_sel2.S`, linker base 0x0e100000, manifest parse, secondary warm-boot, boots SP1+SP2+SP3
- `tfa_boot` (⊃ linux_guest) — SPMC_PRESENT=true, NS proxy forwards DIRECT_REQ/PARTITION_INFO/MEM_SHARE/LEND/RECLAIM to real SPMC via SPMD

**Mutual exclusivity**: `multi_pcpu` ⊥ `multi_vm`; `sel2` ⊥ all others. `tfa_boot` pairs with `run-tfa-linux`.

**Toolchain**: Rust nightly, `aarch64-linux-gnu-{gcc,ar,objcopy}`, `qemu-system-aarch64`.

## Architecture

### Privilege Model
- **EL2** (or **S-EL2** in `sel2`): hypervisor — exceptions, Stage-2 tables, GIC virtual interface
- **EL1**: guest (Linux/Zephyr) or Secure Partition
- **Stage-2**: identity map (GPA==HPA), 2MB blocks + 4KB pages

### Core Abstractions

| Type | File | Role |
|------|------|------|
| `Vm` | `src/vm.rs` | VM lifecycle, Stage-2 setup, `run_smp()` scheduler loop |
| `Vcpu` | `src/vcpu.rs` | State machine, context save/restore |
| `VcpuContext` / `VcpuArchState` | `src/arch/aarch64/regs.rs`, `vcpu_arch_state.rs` | Guest regs; per-vCPU GIC LRs, timer, EL1 sysregs, PAC keys |
| `DeviceManager` | `src/devices/mod.rs` | Enum-dispatch MMIO routing |
| `Scheduler` | `src/scheduler.rs` | Round-robin vCPU scheduler, block/unblock |
| `ExitReason` | `src/arch/aarch64/regs.rs` | VM exit causes (WfiWfe, HvcCall, SmcCall, DataAbort, …) |
| `FfaProxy` | `src/ffa/proxy.rs` | NS-EL2 FF-A v1.1 proxy (VERSION/ID/FEATURES/RXTX/messaging/memory) |
| `Stage2Walker` | `src/ffa/stage2_walker.rs` | Walker from VTTBR_EL2: PTE SW bits, S2AP, map/unmap for cross-VM sharing |
| `FfaDescriptors` | `src/ffa/descriptors.rs` | FF-A v1.1 composite memory region descriptor parsing |
| `PlatformInfo` | `src/dtb.rs` | Runtime DTB parsing (UART/GIC/RAM/CPU discovery) |
| `VSwitch` / `NetRxRing` | `src/vswitch.rs` | L2 vswitch + MAC learning; per-port SPSC RX ring |
| `VirtualPl031` | `src/devices/pl031.rs` | PL031 RTC emulation |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willamhou/hypervisor](https://github.com/willamhou/hypervisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

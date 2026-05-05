---
trigger: always_on
description: NexaOS is a Rust `no_std` hybrid kernel with 6-stage boot (`src/boot/stages.rs`):
---

# NexaOS AI Coding Guide

## Architecture Overview

NexaOS is a Rust `no_std` hybrid kernel with 6-stage boot (`src/boot/stages.rs`): 
**Bootloader → KernelInit → Initramfs → RootSwitch → RealRoot → UserSpace**. 
The kernel runs in Ring 0, userspace in Ring 3 with full POSIX compliance.

### Key Subsystems

| Component | Location | Purpose |
|-----------|----------|---------|
| Boot entry | `src/main.rs` → `src/lib.rs` | Multiboot2 → `kernel_main()`, UEFI → `kernel_main_uefi()` |
| Memory | `src/mm/paging.rs`, `src/process/types.rs` | Identity-mapped kernel, isolated userspace with 4-level paging |
| Scheduler | `src/scheduler/` | **EEVDF algorithm** (Linux 6.6+): vruntime, deadlines, per-CPU queues |
| Syscalls | `src/syscalls/` | 60+ POSIX syscalls, organized by domain (file, process, signal, network, memory, thread, time) |
| Filesystems | `src/fs/initramfs.rs`, `src/fs/` | CPIO initramfs → ext2 rootfs after pivot_root (stage 4) |
| Safety helpers | `src/safety/` | Centralized unsafe wrappers (volatile, MMIO, port I/O, packet casting) |
| Networking | `src/net/` | Full UDP/IPv4 stack, ARP, DNS resolver; TCP in progress |
| Kernel modules | `modules/`, `src/kmod/` | Loadable `.nkm` modules (ext2, e1000, virtio) with PKCS#7 signing |
| Init system | `src/boot/init.rs` | PID 1 service management (System V runlevels, /etc/inittab parsing) |
| NVM Hypervisor | `nvm/` | Enterprise hypervisor platform (VT-x/AMD-V, live migration, HA) |

### Critical Memory Layout (`src/process/types.rs`)

```rust
USER_VIRT_BASE: 0x1000000      // Userspace code base (16MB)
HEAP_BASE:      0x1200000      // User heap (8MB: 0x1200000–0x1A00000)
STACK_BASE:     0x1A00000      // User stack (2MB, placed after heap)
INTERP_BASE:    0x1C00000      // Dynamic linker region (16MB reserved)
```

**⚠️ Critical Invariant**: Changes to these constants require simultaneous updates in:
- `src/mm/paging.rs` (map_user_region, identity mapping)
- `src/process/loader.rs` (ELF loading, segment placement)
- `src/security/elf.rs` (auxiliary vector setup)

Failure to sync these causes memory corruption or segfaults during ELF loading.

### EEVDF Scheduler (`src/scheduler/`)

The scheduler uses EEVDF (Earliest Eligible Virtual Deadline First), same as Linux 6.6+:
- **vruntime**: Tracks weighted CPU time consumption per process
- **Virtual Deadline**: `vruntime + slice/weight` provides latency guarantees
- **Per-CPU queues**: Each CPU has its own run queue to minimize lock contention
- **Eligibility**: Only processes with `lag >= 0` can preempt current task

Key files: `types.rs` (constants), `priority.rs` (vruntime/deadline), `percpu.rs` (per-CPU state)

## Build & Test Workflows

### Standard Commands

```bash
./ndk build full              # ALWAYS START WITH THIS: kernel → nrlib → userspace → modules → rootfs → ISO
./ndk build quick             # Fast: kernel + initramfs + ISO (skip rootfs rebuild)
./ndk build kernel            # Kernel only (use after .rs changes)
./ndk build userspace rootfs iso  # Rebuild after userspace/etc/ changes
./ndk run                     # Boot in QEMU (uses last built ISO)
./ndk dev --quick             # Build + run in one command
./ndk test                    # Run unit tests (tests/ crate)
./ndk test --filter bitmap    # Run specific test pattern
./ndk coverage html           # Generate HTML coverage report
./ndk run --debug             # Start GDB server at 127.0.0.1:1234
```

### Environment Variables

```bash
BUILD_TYPE=debug ./ndk build full     # Debug build (DEFAULT, STABLE)
BUILD_TYPE=release ./ndk build full   # Release (O3 may break fork/exec; avoid)
LOG_LEVEL=info ./ndk build kernel     # Kernel log level: debug|info|warn|error
SMP=8 ./ndk run                       # Boot with 8 CPU cores
MEMORY=2G ./ndk run                   # Boot with 2GB RAM
FEATURE_smp=true ./ndk build kernel   # Enable SMP at build time
```

**Build order is strict**: kernel → nrlib → userspace → modules → initramfs → rootfs → iso.
Skipping steps breaks subsequent builds.

## Coding Conventions

### Kernel Code (`src/`)

- **`no_std` only** — No heap allocations; use fixed-size buffers (StaticVec, ArrayVec)
- **Logging macros** (`src/logger.rs`): `kinfo!`, `kwarn!`, `kerror!`, `kdebug!`, `kfatal!`
  - **Never disable logging** — serial output is essential for boot debugging
  - Log level controlled by kernel command line (e.g., `log_level=debug`)
- **Error handling**: Return `Errno` (from `src/posix.rs`); never panic in syscall paths
- **Unsafe code**: Use `src/safety/` helpers exclusively:
  ```rust
  use crate::safety::{inb, outb, volatile_read, volatile_write, copy_from_user, copy_to_user, cast_header};
  ```
  Rationale: Centralizes x86_64 low-level details in one place for auditing.

### Process & Scheduler Consistency

Process state management is **critical** because three subsystems interact:
1. **Scheduler** (`src/scheduler/mod.rs`) — tracks Ready/Running/Sleeping/Zombie
2. **Signals** (`src/ipc/signal.rs`) — can transition processes to Sleeping/Running
3. **wait4 syscall** (`src/syscalls/process.rs`) — must see consistent Zombie state

**Pattern to follow**:
- Always acquire process lock before modifying `ProcessState`
- After signal delivery, update scheduler queue (don't just change state)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nexa-sys/nexa-os](https://github.com/nexa-sys/nexa-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

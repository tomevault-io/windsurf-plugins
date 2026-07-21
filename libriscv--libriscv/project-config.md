---
trigger: always_on
description: This document explains libriscv from fundamentals to production integration. libriscv is a RISC-V userspace emulator designed for embedding scripting in C++ applications. Host-guest communication has zero overhead under correct configuration because the host reads and writes guest memory directly through pointers, with no serialization or copying. Further, VM call overhead can be as low as ~5ns.
---

# libriscv Architecture Guide

This document explains libriscv from fundamentals to production integration. libriscv is a RISC-V userspace emulator designed for embedding scripting in C++ applications. Host-guest communication has zero overhead under correct configuration because the host reads and writes guest memory directly through pointers, with no serialization or copying. Further, VM call overhead can be as low as ~5ns.

## Table of Contents

1. [Machine Fundamentals](#machine-fundamentals)
2. [Memory Architecture](#memory-architecture)
3. [Configuring the Emulator](#configuring-the-emulator)
4. [Booting and Initialization](#booting-and-initialization)
5. [Host Functions (Syscalls)](#host-functions-syscalls)
6. [Calling Guest Functions](#calling-guest-functions)
7. [Heap Takeover](#heap-takeover)
8. [Guest Datatypes](#guest-datatypes)
9. [Generated Host Functions](#generated-host-functions)
10. [Advanced: Zero-Copy Structures](#advanced-zero-copy-structures)
11. [Production Integration Example](#production-integration-example)
12. [Minimal Script Wrapper (End-to-End)](#minimal-script-wrapper-end-to-end)
13. [Verifying VMCall Latency](#verifying-vmcall-latency)
14. [RPC Between Same-Program Instances](#rpc-between-same-program-instances)

---

## Machine Fundamentals

A `Machine<W>` holds a static RISC-V ELF program. `W` is the pointer width: `4` for RV32, `8` for RV64. The machine contains a CPU with 32 integer registers + 32 float registers + PC, a memory subsystem, an optional native heap arena, and a syscall handler table.

```cpp
#include <libriscv/machine.hpp>
using machine_t = riscv::Machine<8>; // 64-bit RISC-V
using gaddr_t = riscv::address_type<8>;

// Load ELF binary
std::vector<uint8_t> binary = load_file("program.elf");
machine_t machine(binary, {
    .memory_max = 64ULL << 20,  // 64 MB
    .stack_size = 1ULL << 20,   // 1 MB
});
```

### Constructor Variants

| Constructor | Use Case |
|---|---|
| `Machine(binary, opts)` | Normal: load ELF from binary data |
| `Machine(other_machine, opts)` | Forking: CoW clone for isolation or threading |
| `Machine(opts)` | Empty machine (no program, manual setup) |

The forking constructor creates a thin copy-on-write clone. The parent machine must outlive all forks. Forks share read-only pages and the decoded execute segment, consuming minimal memory. A full fork-execute-destruct cycle completes in under a microsecond, but *requires* virtual paging and *cannot work* with a flat arena.

### Execution Model

There is no on/off state. It is just memory and registers. Execution happens when you call one of the simulation functions, which run instructions until a stopping condition is met:

| Function | Stops when | Use case |
|---|---|---|
| `machine.simulate(max)` | Instruction counter reaches `max`, or STOP is hit | Booting `main()` |
| `machine.vmcall<MAX>(addr, args...)` | Instruction counter reaches `MAX`, or guest function returns | Calling guest functions with a timeout |
| `machine.vmcall<0>(addr, args...)` | Guest function returns (no counter check) | Calling guest functions without a timeout |
| `cpu.simulate_inaccurate(pc)` | STOP is hit (no counter check) | Lowest-overhead execution |

When the `vmcall` template parameter is `0` or `UINT64_MAX`, it internally uses `CPU::simulate_inaccurate()`, a fast-path that skips instruction counting entirely. No counter increment, no limit comparison on every instruction. This is often 15% faster than with instruction counting.

The instruction counter exists to catch runaway execution — infinite loops, not security exploits. A guest stuck in a loop cannot steal data or escape the sandbox; it can only hog CPU time. Instruction counting is most useful during development when guest code might get stuck, or in environments where it is important that no single call runs too long. Syscall handlers can add penalties to the counter (e.g., `malloc` adds thousands of instructions) to account for host-side work that the counter wouldn't otherwise see.

Between calls, the machine retains all state: registers, memory, stack, heap. You call `vmcall`, it runs, the guest function returns (hitting the exit function), and the machine is immediately ready for the next call. There is no teardown or re-initialization.

---

## Memory Architecture

libriscv has two memory modes. The choice affects every subsequent API decision.

### Flat Read-Write Arena (default, recommended)

All guest memory lives in one contiguous host allocation. Memory access is a bounds check + pointer dereference. This enables:

- **Direct pointer access**: `machine.memory.memarray<T>(addr, count)` returns a `T*` into guest memory. No copy. You read and write through it.
- **String views**: `machine.memory.memview(addr, len)` returns a `std::string_view` of guest memory. No copy.
- **Span access**: `machine.memory.memspan<T>(addr, count)` returns a `std::span<T>`. No copy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libriscv/libriscv](https://github.com/libriscv/libriscv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

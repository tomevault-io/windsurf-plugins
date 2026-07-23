---
trigger: always_on
description: > For AI coding assistants. Read this before making changes.
---

# Profile-Bee Developer Guide

> For AI coding assistants. Read this before making changes.

## What Is This?

An eBPF-based CPU profiler for Linux, written in Rust. Single binary (`probee`), no BCC/libbpf dependencies. Walks stacks in-kernel via frame pointers or DWARF unwind tables, outputs flamegraphs (TUI, SVG, HTML, JSON, web server).

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  profile-bee-ebpf (kernel)     profile-bee (userspace)          │
│  ─────────────────────────     ───────────────────────          │
│  perf_event fires              tokio async runtime              │
│    → collect_trace()             ← RingBuf polling              │
│    → FP or DWARF unwinding       → read COUNTS, STACK_TRACES,  │
│    → write to eBPF maps            STACKED_POINTERS maps        │
│    → submit StackInfo via        → symbolize (blazesym/gimli)   │
│      RING_BUF_STACKS            → output flamegraph             │
│                                                                 │
│  profile-bee-common (#![no_std])                                │
│  ────────────────────────────────                               │
│  Shared #[repr(C)] types: StackInfo, FramePointers,             │
│  UnwindEntry, ProcInfo, DwarfUnwindState, constants             │
└─────────────────────────────────────────────────────────────────┘
```

## Crate Layout

| Crate | What | Notes |
|-------|------|-------|
| `profile-bee/` | Userspace binary (`probee`, `pbee`) | CLI, eBPF loader, symbolization, output generation |
| `profile-bee-ebpf/` | eBPF kernel programs | **Separate workspace** — built with `cargo +nightly` for `bpfel-unknown-none` |
| `profile-bee-common/` | Shared `#![no_std]` types | `#[repr(C)]` structs used by both eBPF and userspace |
| `profile-bee-tui/` | Terminal UI (forked from flamelens) | Behind `tui` feature flag (default on) |
| `xtask/` | Build automation | `cargo xtask build-ebpf`, `cargo xtask run` |

## Key Files — Read Order

| # | File | Lines | What |
|---|------|-------|------|
| 1 | `profile-bee-common/src/lib.rs` | ~190 | All shared types and constants (`StackInfo`, `UnwindEntry`, `FramePointers`, `ProcInfo`, `DwarfUnwindState`) |
| 2 | `profile-bee-ebpf/src/main.rs` | ~110 | All eBPF program entry points — thin wrappers calling into `lib.rs` |
| 3 | `profile-bee-ebpf/src/lib.rs` | ~1200 | eBPF core: `collect_trace`, DWARF unwinding, all map definitions |
| 4 | `profile-bee/src/ebpf.rs` | ~500 | `EbpfProfiler` — eBPF loading, program attachment, map setup, DWARF table loading |
| 5 | `profile-bee/src/dwarf_unwind.rs` | ~880 | `DwarfUnwindManager` — `.eh_frame` parsing, unwind table generation, shard management |
| 6 | `profile-bee/src/trace_handler.rs` | ~290 | `TraceHandler` — symbolization (blazesym), stack frame formatting |
| 7 | `profile-bee/bin/profile-bee.rs` | ~1750 | CLI (`Opt` struct via clap), orchestration, all output modes, main loop |
| 8 | `profile-bee/src/probe_spec.rs` | ~780 | `ProbeSpec` / `SymbolPattern` — uprobe spec parsing (glob, regex, demangled, source loc) |
| 9 | `profile-bee/src/probe_resolver.rs` | ~670 | `ProbeResolver` — resolves `ProbeSpec` to concrete `ResolvedProbe` targets via ELF scanning |

## Module Map — Userspace (`profile-bee/`)

```
bin/profile-bee.rs          CLI entry point, Opt (clap), main loop, output writers
src/
  lib.rs                    Re-exports; declares modules
  ebpf.rs                   EbpfProfiler: load/attach eBPF, map I/O, DWARF table loading
  dwarf_unwind.rs           DwarfUnwindManager: .eh_frame → UnwindEntry tables → eBPF shards
  trace_handler.rs          TraceHandler: address → symbol resolution (blazesym), caching
  types.rs                  StackFrameInfo, FrameCount, StackInfoExt trait
  cache.rs                  ProcessCache, AddrCache, PointerStackFramesCache (LRU-style)
  html.rs                   HTML flamegraph output (Stack tree builder)
  spawn.rs                  SpawnProcess: child process lifecycle for --cmd
  probe_spec.rs             ProbeSpec enum: parse uprobe specs (exact/glob/regex/source/demangled)
  probe_resolver.rs         ProbeResolver: spec → ResolvedProbe via ELF .symtab/.dynsym scanning
  legacy/                   Legacy symbol resolution code (addr2line-based, being replaced by blazesym)
```

## Module Map — eBPF (`profile-bee-ebpf/`)

```
src/main.rs                 Entry points: #[perf_event], #[kprobe], #[uprobe], etc.
src/lib.rs                  All implementation + map definitions:
                              collect_trace()              — main sampling path (FP + optional DWARF)
                              collect_trace_stackid_only() — tracepoint path (bpf_get_stackid only)
                              collect_trace_raw_syscall()  — raw_tracepoint sys_enter
                              collect_trace_raw_tp_with_task_regs() — raw TP with bpf_task_pt_regs
                              dwarf_unwind_one_frame()     — single DWARF frame step
                              dwarf_copy_stack_regs()      — legacy inline path (21 frames max)
                              dwarf_try_tail_call()        — tail-call path entry (165 frames max)
                              dwarf_unwind_step_impl()     — tail-call target (5 frames per call)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zz85/profile-bee](https://github.com/zz85/profile-bee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

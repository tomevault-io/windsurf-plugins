---
trigger: always_on
description: **Detect which repo you're in from the directory name:**
---

# Monk OS - Agent Instructions & Technical Architecture

## Clone Context

**Detect which repo you're in from the directory name:**

| Directory | Role | Rules |
|-----------|------|-------|
| `os/` | **Main** | Stable. No speculative changes. Run tests before committing. |
| `os-dev*/` | **Development** | Active work. Ask user for current goal if unclear. |

**In main**: Be conservative. Only well-tested, complete changes.

**In dev clones**: Move fast, experiment freely. The user will direct you.

---

## Prompt Library (`prompts/`)

**Select and apply prompts based on what you're working on:**

| File | Use When | Summary |
|------|----------|---------|
| `planning-mode.md` | Designing new features | Collaborative planning via `docs/`. Plans persist across sessions, iterate with user. |
| `kernel-dev.md` | Modifying `src/` (kernel, HAL, VFS, EMS) | Linux kernel + TypeScript staff engineer code review. Focus on race conditions, invariants, concurrency, testability. |
| `userspace-dev.md` | Modifying `rom/` (bin commands, lib utilities) | GNU coreutils + TypeScript code review. Focus on POSIX compatibility, streams, error handling, argument parsing. |
| `parallel-agents.md` | Bulk changes across many files | How to split work across 2-5 parallel agents. ~4x faster than sequential. |
| `tester.md` | Fixing `spec/` or `perf/` after refactors | Parallel agent strategy for fixing typecheck and test failures. |
| `performance.md` | Writing `perf/` tests | Throughput, stress, and backend comparison tests. Run via `bun run perf`. |

**Before major work**: Read the relevant prompt and apply its standards.

**For large changes (5+ files)**: Use `parallel-agents.md` to split work. Parallel agents have been very successful for bulk refactors and conversions.

---

> **Quick Context**: Monk OS reframes API architecture as an operating system where **Bun is the hardware**. The single-executable deployment (`bun build --compile`) isn't packaging an app—it's burning firmware.

## 1. Core Philosophy & Architecture

### System Design Principles
- **Everything is a file**: Uniform namespace following Plan 9's paradigm
- **Files are queryable**: BeOS-style database-centric filesystem (files have UUIDs, indexed)
- **Process isolation**: Each process is a Bun Worker with isolated memory
- **Message-driven**: All internal communication uses structured `Message` and `Response` objects
- **Streams-first**: Default API is `AsyncIterable<Response>`, not arrays

### Layered Architecture
```
┌─────────────────────────────────────────────────────────────┐
│  External Applications (os-sdk via Unix socket)             │
├─────────────────────────────────────────────────────────────┤
│  Gateway (src/gateway/) - MessagePack wire protocol         │
├─────────────────────────────────────────────────────────────┤
│  OS Public API (src/os/)                                    │
│  ├── OS class (boot, exec, shutdown, syscall wrappers)      │
│  ├── Domain wrappers (ems, vfs, process)                    │
│  ├── Convenience helpers (read, text, spawn, mount, copy)   │
│  └── Service management (start, stop, restart, list)        │
├─────────────────────────────────────────────────────────────┤
│  Dispatch Layer (src/dispatch/)                             │
│  ├── Dispatcher (syscall routing + sigcall routing)         │
│  ├── StreamController (backpressure, ping/cancel protocol)  │
│  ├── Sigcall registry (userspace handler registration)      │
│  ├── Domain handlers (vfs, ems, hal, process, handle, pool) │
│  └── Worker message handling (onWorkerMessage callback)     │
├─────────────────────────────────────────────────────────────┤
│  Kernel Layer (src/kernel/)                                 │
│  ├── Process Manager (Worker lifecycle, signals)            │
│  ├── Handle System (file, socket, pipe, port, channel)      │
│  ├── Worker Pools (PoolManager, auto-scaling)               │
│  ├── Service Activation (boot, tcp, udp, pubsub, watch)     │
│  └── Extracted functions (kernel/kernel/ subdirectory)      │
├─────────────────────────────────────────────────────────────┤
│  VFS - Virtual File System (src/vfs/)                       │
│  ├── Path resolution, mount table, model coordination       │
│  └── Models: File, Folder, Device, Proc, Link               │
├─────────────────────────────────────────────────────────────┤
│  EMS - Entity Model System (src/ems/)                       │
│  ├── Database abstraction (SQLite, PostgreSQL)              │
│  ├── Observer pipeline (8 rings for mutation processing)    │
│  ├── EntityOps, ModelCache, EntityCache                     │
│  └── Streaming queries with backpressure                    │
├─────────────────────────────────────────────────────────────┤
│  HAL - Hardware Abstraction (src/hal/)                      │
│  ├── BlockDevice, StorageEngine, NetworkDevice, FileDevice  │
│  ├── Timer, Clock, Entropy, Crypto, Console                 │
│  ├── DNS, Host, IPC, Channel, Compression                   │
│  └── BunHAL implementation                                  │
├─────────────────────────────────────────────────────────────┤
│  Bun Runtime (Host OS, Workers, primitives)                 │
└─────────────────────────────────────────────────────────────┘
```

### Naming Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ianzepp/monk-os-kernel](https://github.com/ianzepp/monk-os-kernel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->

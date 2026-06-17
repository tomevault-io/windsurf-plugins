---
trigger: always_on
description: > **Quick Reference:** This document is designed for AI assistants (Claude, GPT, etc.) working on AethelOS. It provides build commands, design philosophy, coding standards, and pointers to architectural documentation.
---

# CLAUDE.md - AI Assistant Guide to AethelOS Development

> **Quick Reference:** This document is designed for AI assistants (Claude, GPT, etc.) working on AethelOS. It provides build commands, design philosophy, coding standards, and pointers to architectural documentation.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Design Philosophy](#design-philosophy)
3. [Architecture Overview](#architecture-overview)
4. [Coding Standards & Quality Controls](#coding-standards--quality-controls)
5. [Documentation Index](#documentation-index)
6. [Current Status & Roadmap](#current-status--roadmap)
7. [Common Tasks](#common-tasks)

---

### Build Commands

**From project root:**

```bash
# 1. Build the kernel and ISO

cd /f/OS/user_programs/hello && cargo build --release --target x86_64-unknown-none 2>&1 | grep -E "(Compiling|Finished|error)"
cd /f/OS && cargo build 2>&1 | grep -E "(Compiling heartwood|Finished|error)" && wsl bash -c "cp target/x86_64-aethelos/debug/heartwood isodir/boot/aethelos/heartwood.bin && grub-mkrescue -o aethelos.iso isodir 2>&1 | grep -E '(success|Writing)'"


# 2. Run in QEMU
Do not attempt to run in QEMU yourself. Pass the turn to the human after a build and alert them to run it for you and inform you of the results.



### Prerequisites

- **Rust:** Nightly toolchain (configured in `rust-toolchain.toml`)
  - Components: `rust-src`, `llvm-tools-preview`
- **GRUB:** `grub-mkrescue` (via WSL on Windows, native on Linux)
- **QEMU:** `qemu-system-x86_64` for testing

---

## Design Philosophy

### Core Principle: Symbiotic Computing

> *"The code does not command the silicon. The silicon does not serve the code. They dance together, and in that dance, life emerges."*

AethelOS is **not** a clone of Unix, Linux, or Windows. It is a ground-up rethinking of operating system design based on these principles:

### 1. **Harmony Over Force**

- **NOT:** Preemptive scheduling that interrupts processes arbitrarily
- **BUT:** Cooperative negotiation where processes yield willingly
- **METAPHOR:** The Loom of Fate weaves threads together harmoniously

**Implementation:**
- Threads have harmony scores based on CPU usage, yield frequency, and cooperation
- "Parasitic" threads are throttled (slowed), not killed
- System-wide harmony metrics guide scheduling decisions

### 2. **Memory Over Forgetting**

- **NOT:** Files that are overwritten and lost
- **BUT:** Git-like versioning built into the filesystem
- **METAPHOR:** The World-Tree remembers all versions like tree rings

**Implementation:**
- Content-addressable storage (SHA-256)
- Global commit graph tracking all changes
- Query-based file discovery: `seek scrolls where essence is "Scroll" and creator is "Elara"`
- Intelligent pruning prevents unbounded growth

### 3. **Beauty as Necessity**

- **NOT:** Aesthetics as afterthought
- **BUT:** Visual design reveals system state intuitively
- **METAPHOR:** The system's appearance is its truth

**Implementation:**
- Poetic naming (Loom of Fate, Mana Pool, World-Tree)
- Unicode symbols (◈ for emphasis, tree metaphors)
- VGA graphics mode planned for vector-based GUI
- Color-coded thread states, visual harmony indicators

### 4. **Security Through Nature**

- **NOT:** Access control lists and permissions bits
- **BUT:** Unforgeable capability tokens
- **METAPHOR:** Natural boundaries, not artificial walls

**Implementation:**
- Capability-based security (no raw pointers in userspace)
- Capabilities grant specific rights (read, write, execute, delegate)
- Capabilities can be attenuated (reduced permissions) but never amplified
- Hardware MMU enforces capability boundaries
- Borrowing ideas from known good kernel hardening efforts, grsec/PaX etc.
---

## Architecture Overview

### High-Level Structure

```
┌─────────────────────────────────────────────────────────┐
│                    User Space                           │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │ Eldarin     │  │ User Apps    │  │ System Utils  │  │
│  │ Shell       │  │ (Future)     │  │               │  │
│  └─────────────┘  └──────────────┘  └───────────────┘  │
└─────────────────────────────────────────────────────────┘
                          ↕ (Capability-based syscalls)
┌─────────────────────────────────────────────────────────┐
│                    Groves (Services)                     │
│  ┌──────────────┐ ┌──────────────┐ ┌────────────────┐  │
│  │ World-Tree   │ │ The Weave    │ │ Network Sprite │  │
│  │ (Filesystem) │ │ (Compositor) │ │ (Networking)   │  │
│  └──────────────┘ └──────────────┘ └────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          ↕ (IPC via Nexus)
┌─────────────────────────────────────────────────────────┐
│              Heartwood (Kernel)                         │
│  ┌──────────────┐ ┌──────────────┐ ┌────────────────┐  │
│  │ Loom of Fate │ │ Mana Pool    │ │ Attunement     │  │
│  │ (Scheduler)  │ │ (Memory)     │ │ (Hardware)     │  │
│  └──────────────┘ └──────────────┘ └────────────────┘  │
│  ┌──────────────┐ ┌──────────────┐                     │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dollspace-gay/AethelOS](https://github.com/dollspace-gay/AethelOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

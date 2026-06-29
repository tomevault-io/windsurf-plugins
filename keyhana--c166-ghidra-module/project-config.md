---
trigger: always_on
description: > **Purpose**: This document provides comprehensive context for AI agents working on this Ghidra extension for the Infineon C166 microcontroller architecture.
---

# C166 Ghidra Module - Agent Documentation

> **Purpose**: This document provides comprehensive context for AI agents working on this Ghidra extension for the Infineon C166 microcontroller architecture.

---

## ⚠️ Important: Instruction Set Reference

**Before modifying, verifying, or implementing any C166 instruction**, consult:

```
c166ism.md
```

This file contains the complete **C166 Instruction Set Manual** (converted from PDF). It includes:
- Instruction syntax and operation semantics
- Condition flag behavior (E, Z, V, C, N) for each instruction
- Addressing modes and encoding formats
- Detailed descriptions and edge cases

**Always verify instruction behavior against this reference** before making changes to `c166.sinc`.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Directory Structure](#directory-structure)
3. [C166 Memory Model](#c166-memory-model)
4. [PCode Injection System](#pcode-injection-system)
5. [Switch Table Handling](#switch-table-handling)
6. [Analyzers](#analyzers)
7. [Ghidra Scripts](#ghidra-scripts)
8. [Edge Cases and Gotchas](#edge-cases-and-gotchas)
9. [Build and Development](#build-and-development)
10. [Known Limitations](#known-limitations)

---

## Architecture Overview

The C166 is a **16-bit microcontroller** with a **24-bit address space**. The key challenge is that C166 uses a **segmented memory model** where 16-bit pointers are translated to 24-bit physical addresses using **Data Page Pointers (DPP)** or **Extended Page/Segment (EXTP/EXTS)** registers.

### Key Architectural Features

| Feature | Description |
|---------|-------------|
| **Word Size** | 16-bit |
| **Address Space** | 24-bit (16MB) |
| **Pointer Size** | 16-bit (paged) or 24-bit (far) |
| **DPP Registers** | DPP0-DPP3 (10-bit page values) |
| **Paging Formula** | `physical = (DPP << 14) | (offset & 0x3FFF)` |

---

## Directory Structure

```
GhidraInfineon/
├── data/
│   └── languages/
│       ├── c166.slaspec       # Main SLEIGH entry point
│       ├── c166.sinc          # SLEIGH instruction definitions
│       ├── c166.cspec         # Compiler specification (calling conventions, callotherfixup)
│       ├── c166.ldefs         # Language definitions
│       ├── c166cr.pspec       # Processor spec for C167CR (includes segmentop)
│       ├── c167cs.pspec       # Processor spec for C167CS (includes segmentop)
│       └── c166.sla           # Compiled SLEIGH (auto-generated, delete to rebuild)
├── src/main/java/ghidrainfineon/
│   ├── C166AddressAnalyzer.java   # DPP-aware address resolution analyzer
│   ├── PcodeInject.java           # PCode injection library registration
│   ├── GetPagedOffset.java        # Injector for static address paging
│   ├── SwitchLoad.java            # Injector for switch table loads
│   └── PcodeOpEmitter.java        # Helper for emitting PCode operations
├── ghidra_scripts/
│   ├── CreateDPPReference.java    # Manual DPP reference creation script
│   └── C166SwitchOverride.java    # Manual switch table override script
├── build.gradle                   # Gradle build configuration
└── extension.properties           # Ghidra extension metadata
```

### File Purposes

| File | Purpose |
|------|---------|
| **c166.sinc** | Defines all C166 instructions, registers, and pcodeops. This is where `GetPagedOffset`, `segment`, and `c166_switch_load` are defined. |
| **c166.cspec** | Defines `callotherfixup` entries that map pcodeops to Java injectors. The `dynamic="true"` attribute enables runtime PCode injection. |
| **c166cr.pspec / c167cs.pspec** | Defines `segmentop` which tells the decompiler how to interpret `segment()` PCode for address calculation. |
| **PcodeInject.java** | Registers custom PCode injectors (`GetPagedOffset`, `SwitchLoad`) with Ghidra's language system. |

---

## C166 Memory Model

### DPP (Data Page Pointer) Translation

C166 uses a 4-page model for data access:

```
16-bit offset: [15:14] = DPP selector, [13:0] = page offset

DPP0: offsets 0x0000-0x3FFF
DPP1: offsets 0x4000-0x7FFF  
DPP2: offsets 0x8000-0xBFFF
DPP3: offsets 0xC000-0xFFFF

Physical address = (DPP[selector] << 14) | (offset & 0x3FFF)
```

### EXTP/EXTS Override

The `EXTP` and `EXTS` instructions temporarily override DPP translation:

| Instruction | Behavior | Formula |
|-------------|----------|---------|
| **EXTP** | Page-based override | `(Extp << 14) | (offset & 0x3FFF)` |
| **EXTS** | Segment-based override | `(Exts << 16) | (offset & 0xFFFF)` |

**Critical**: EXTS uses the **full 16-bit offset**, while EXTP uses only the **lower 14 bits**.

### Code Addressing

Code addresses use a different model - `jmpi` uses the current code segment:

```
jmpi [rX]  →  target = (current_IP & 0xFF0000) | rX_value
```

---

## PCode Injection System

### How It Works

1. **SLEIGH** (`c166.sinc`) defines `pcodeop` placeholders:
   ```sleigh
   define pcodeop GetPagedOffset;
   define pcodeop segment;
   define pcodeop c166_switch_load;
   ```

2. **Compiler Spec** (`c166.cspec`) maps pcodeops to injectors:
   ```xml
   <callotherfixup targetop="c166_switch_load">
       <pcode dynamic="true">
           <input name="ptr" size="2"/>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keyhana/c166-ghidra-module](https://github.com/keyhana/c166-ghidra-module) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

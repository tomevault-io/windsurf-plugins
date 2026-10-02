---
trigger: always_on
description: Working instructions for all contributors and coding agents working in this repository. Read this file before performing any task.
---

# P3P3DS — Engineering Rules & Working Notes

Working instructions for all contributors and coding agents working in this repository. Read this file before performing any task.

---

## 1. What is P3P3DS

**Objective:** Run *Persona 3 Portable* (`ULUS-10512`, North American release) on the **New Nintendo 3DS / New 3DS XL / New 2DS XL** family of consoles.

**Working Hypothesis (Architecture C: Hybrid Static Recompilation):**
- **P3P Allegrex code:** Ahead-of-Time (AOT) offline static recompilation into C/C++ translation units, compiled to native ARM11 machine code via devkitARM GCC.
- **PSP OS & Kernel services:** Lightweight, modular High-Level Emulation (HLE) runtime (C/C++).
- **PSP Graphics Engine (GE):** Native `citro3d` / DMP PICA200 hardware renderer backend.
- **PSP Audio:** Native Nintendo 3DS DSP (`ndsp`) hardware-accelerated audio backend.
- **PSP Filesystem:** Multi-tier Virtual File System (VFS) with runtime mod and asset redirection on SDMC (`sdmc:/p3p3ds/`).

*The architecture is not a dogma:* If empirical experiments or profiling demonstrate that an aspect (e.g. interpreter fallback for unresolved indirect calls, or an alternative renderer pipeline) is superior, document the proof with benchmarks and propose the revision.

---

## 2. Workspace Containment

All temporary files, caches, build output, and generated data produced by the project must reside strictly inside the root of the P3P3DS workspace:
- **Never use external directories** such as `C:\`, user home (`~`), `%TEMP%`, `%APPDATA%`, `%LOCALAPPDATA%`, Documents, Desktop, or other host paths without explicit user permission.
- **Allowed workspace directories:**
  - `.tmp/` — Temporary test files, intermediate script scratchpads.
  - `.cache/` — Build and tool caches.
  - `build/` — Intermediate build artifacts.
  - `out/` — Final binary artifacts.
- **Environment redirection:** Where possible before running tests or compilers, redirect `TEMP`, `TMP`, and `TMPDIR` to `.tmp/` within the workspace.
- **Host environment integrity:** Never install global dependencies, modify system PATH, touch the Windows registry, or edit global user configs without explicit user authorization.

---

## 3. Core Engineering Principle: MEASURE FIRST

Never make unverified technical claims. Do not write phrases such as:
- *"will run at 60 FPS"*
- *"zero cost abstraction"*
- *"native speed"*
- *"1:1 hardware mapping"*
- *"JIT is impossible"*
- *"this API is completely unused"*

without providing concrete measurements, source code citations, disassembly excerpts, or reproducible benchmarks.

**Claim Status Markings:**
Every non-trivial architectural or technical assertion in documentation and reports must be tagged:
- `[VERIFIED]`: Confirmed against real game executable, hardware test, or reference code (must cite exact repository, file, and line/symbol).
- `[INFERRED]`: Strongly suggested by architecture or patterns, but not yet directly measured on target hardware.
- `[UNVERIFIED]`: Working hypothesis or theoretical assumption requiring empirical testing.
- `[WRONG]`: Previously believed claim refuted by empirical test or source inspection (keep in log to prevent regressions).

---

## 4. Source Hierarchy

Establish the observed guest behavior first. For PSP API semantics, research in this order when available:
**uOFW -> PSPSDK -> pspautotests -> PPSSPP (behavioral reference only).**
Do not copy PPSSPP implementation code. Record disagreements rather than silently selecting convenient behavior; direct game/hardware evidence constrains the implementation.

For evidence provenance (not a competing API research order):

1. **Real P3P Executable (`ULUS-10512`) / Live Runtime Observations**
2. **PSP Hardware Tests / `references/pspautotests`**
3. **uOFW (`references/uofw`) and PSPSDK (`psp/pspsdk`) contracts**
4. **PPSSPP (`references/ppsspp`) behavioral reference, not implementation source**
5. **PSPRecomp (`recomp/PSPRecomp`) & Yakumo (`recomp/Yakumo`) Observed Behaviors**
6. **Existing P3P Community Patches & Reverse Engineering (`p3p/p3p-patches`, Mod Menu)**
7. **General Internet / Forum Documentation**
8. **Hypotheses & Assumptions**

*Never present an assumption as a verified fact.*

---

## 5. Repository Boundaries & Third-Party Code

The following directories contain upstream/reference projects and must **never** undergo mass refactoring or formatting sweeps:
- `recomp/` (PSPRecomp, Yakumo, sal063-recomp, psprecomp, N64Recomp)
- `references/` (ppsspp, pspautotests, uofw, DaedalusX64-3DS)
- `psp/` (pspsdk, vfpu-docs, prxtool, ghidra-allegrex)
- `3ds/` (libctru, citro3d, citro2d, 3ds-examples)
- `p3p/` (p3p-patches, Persona-3-Portable-Mod-Menu)
- `tools/` (Atlus-Script-Tools, AemulusModManager, Amicitia, AtlusFileSystemLibrary, CriFsV2Lib, CriPakTools)

**Our Code Boundaries:**
All P3P3DS-specific code lives in:
- `core/` (Target-agnostic P3P runtime, HLE definitions, memory map)
- `platform/pc/` (Development/debugging PC host runner)
- `platform/3ds/` (New 3DS `libctru`/`citro3d`/`ndsp` native implementation)
- `profiles/p3p/` (P3P-specific static recompilation profile, configs, generated units)
- `experiments/` (Isolated standalone microtests)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [endercodezz/P3P3DS](https://github.com/endercodezz/P3P3DS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

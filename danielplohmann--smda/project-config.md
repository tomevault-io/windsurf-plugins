---
trigger: always_on
description: Guidance for AI coding agents (and humans) working in this repository. Follow this before making changes.
---

# AGENTS.md

Guidance for AI coding agents (and humans) working in this repository. Follow this before making changes.

## Project Overview

SMDA is a minimalist recursive disassembler optimized for accurate Control Flow Graph (CFG) recovery from memory dumps and raw files. It is built on [Capstone](https://www.capstone-engine.org/) and supports:

- **intel** — x86 / x64
- **aarch64** — ARM64
- **cil** — .NET / CIL (via `dncil` / `dnfile`)
- **dalvik** — Android DEX bytecode

**Backend maturity:** `intel` is production-ready and rivals other disassemblers in accuracy. `aarch64` (ARM64) is a newer but consistent addition — on test data it matched IDA's output closely, so treat it as more mature than the rest. `cil` (.NET) and `dalvik` are recent additions that produce solid results but have **not yet been benchmarked against non-uniform / obfuscated code**, so expect caveats.

Inputs are arbitrary memory dumps (ideally with a known base address), raw DEX files, or binary files parsed via LIEF.
Output is a tree of functions → basic blocks → instructions with inter-block / inter-function edges. On top of the recovered CFG, SMDA performs a variety of optional recovery passes: symbol extraction and reconstruction (e.g. Windows API references via the ApiScout method, exports, imports, ELF/PE symbols, Rust/Delphi VMT parsing), and referenced string parsing. A function may only contain instructions that belong to a single function, and instructions may not overlap (IDA-style model).

Key entry point: `smda.Disassembler.Disassembler`.

## Disassembly Pipeline

SMDA is a **recursive (descent) disassembler** organized into phases, designed to recover code robustly from memory dumps with minimal assumptions about context. The guiding principle is aggressive, completeness-over-accuracy **function entry point (FEP)** detection.

> **Scope:** Phases 1–2 below describe the **intel (x86/x64)** pipeline in detail and are the reference model. Other backends implement the same `ArchBackend` contract but may differ substantially — e.g. `cil` (.NET) typically performs **no gap search at all** because the rich metadata already yields accurate FEPs, while `aarch64`/`dalvik` follow their own candidate and traversal logic. Do not assume the intel heuristics (opcode patterns, prologues, NOP lists) apply verbatim to other architectures.

### Phase 1 — Function Entry Point discovery & disassembly (intel)

A single scan over the input buffer locates two kinds of FEP candidates:

- **Code references:** scan for call/jump opcode byte patterns (e.g. `E8` relative call on x86, `FF 15`/`FF 25` RIP-relative on x64) and compute their targets. Any target landing inside the buffer becomes a candidate.
- **Common prologues:** match a small, deliberately conservative set of stack-frame / hotpatch prologues (e.g. `55 8B EC`, `8B FF 55 8B EC`).

When an unmapped PE/ELF is given (not a bare dump), LIEF provides additional candidates from the entry point and exports; candidates outside executable sections are discarded. Candidates are scored by reference count and prologue match, sorted, then disassembled one function at a time via recursive descent (DFS over basic-block starts):

- **Call:** does not end the block; used to update the FEP queue. Register-indirect calls are resolved via local dataflow backtracking.
- **Unconditional jump:** ends the block; target added to the work stack. Jump tables (switch) are resolved via a dedicated heuristic; jumps into a known FEP are treated as tailcalls.
- **Conditional jump / loop:** ends the block; fall-through added first, then the branch/loop target.
- **Terminating (`ret`/`int3`):** ends the function/block. A `push`-`ret` obfuscation construct is recognized and its target resolved.

FEP candidates outside executable regions are skipped; the FEP queue is continuously updated as indirect calls are resolved.

### Phase 2 — Gap analysis (intel)

After Phase 1, the spaces between recovered function bodies are scanned linearly for **gap functions** (unreferenced or unresolved functions, important for completeness). Within gaps:

- **Effective NOPs** (including padding `int3`/`CC` for alignment) are skipped.
- A non-NOP instruction is tested against a short list of common function-start bytes; on match, a recursive-disassembly attempt is made. Success extends the gap past the new function; failure moves to the next gap.

When an unmapped PE/ELF is given, gap search is limited to executable sections. Because FEP detection + gap analysis is intentionally aggressive, it can produce false positives — accepted because completeness is prioritized over precision.

### Phase 3 — Report finalization, escaping & hashing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielplohmann/smda](https://github.com/danielplohmann/smda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

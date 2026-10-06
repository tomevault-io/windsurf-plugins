---
trigger: always_on
description: Matching decompilation of SSBU (Nintendo Switch, AArch64) targeting patch 13.0.4 (final version).
---

# SSBU Decomp — Matching Decompilation of Super Smash Bros. Ultimate

## Project Overview
Matching decompilation of SSBU (Nintendo Switch, AArch64) targeting patch 13.0.4 (final version).

## Target Binary
- **Game**: Super Smash Bros. Ultimate v13.0.4
- **Platform**: Nintendo Switch (NX64 / AArch64)
- **Engine**: cross2app (Bandai Namco proprietary)
- **SDK**: NintendoSDK 8.2.1
- **Build path**: `C:/c2/p4/patch-13.0.1/work/build/vs2015.built/cross2app+standalone/NX64/Release/`
- **Binary format**: NSO (compressed ELF)
- **.text size**: ~60 MB
- **Total functions**: 39,635 (9,298 named by community)

## Compiler
- NX Clang (Nintendo's proprietary Clang/LLVM fork)
- SDK 8.2.1 → **upstream Clang 8.0.0** (confirmed via KinokoDecomp-S)
- Installed at `C:\llvm-8.0.0\`
- Key flags: `-target aarch64-none-elf -mcpu=cortex-a57 -O2 -fPIC -mno-implicit-float -fno-strict-aliasing -fno-slp-vectorize -DMATCHING_HACK_NX_CLANG`
- Post-processing: `tools/fix_orr_movz.py` patches `orr Wd, wzr, #imm` → `movz Wd, #imm`
- Known NX fork divergence: prologue frame pointer scheduling in non-leaf functions
- Use `asm("")` barriers (guarded by `MATCHING_HACK_NX_CLANG`) for instruction ordering issues

## Project Structure
- `src/` — Decompiled C++ source files (44 files)
- `include/` — Headers (game structs, SDK types)
- `asm/` — Expected assembly from Ghidra (for diffing)
- `tools/` — Build/diff/verification scripts
- `config/` — Symbol maps, first targets
- `data/` — Function database (functions.csv with 39,635 entries)

## Build
```bash
python tools/build.py
```
Or: `python tools/progress.py` to check status.
**WARNING:** `build.bat` is deprecated — it runs obsolete post-processors that inflate match counts. Always use `build.py`.

## Module Accessor Pattern
Most `lua_bind` functions take `BattleObjectModuleAccessor*` as first param and dispatch to a sub-module via vtable. The accessor has module pointers at fixed offsets (+0x38 through +0x188). See `include/app/BattleObjectModuleAccessor.h` for the full layout.

Key module offsets: +0x38 Posture, +0x40 Status, +0x48 Control, +0x50 Work, +0x58 Ground, +0x60 Camera, +0x68 Kinetic, +0x88 Motion, +0xA0 Attack, +0xA8 Damage, +0xC0 Area, +0x140 Effect, +0x148 Sound.

## Conventions
- Function names use Skyline/ultimate-research modding community naming where available
- Use `app::lua_bind` namespace for all decompiled functions
- `BattleObjectModuleAccessor*` is the standard first param for module functions
- FighterManager, FighterInformation, BattleObjectSlow/World operate on their own struct pointers (not accessor)
- Vtable offset macros: `#define VT(m) (*reinterpret_cast<void***>(m))`

## Multi-Agent Parallel Workflow
This repo supports parallel decomp work via git worktrees. Multiple Claude Code sessions can work simultaneously on different modules.

### Architecture
- **Orchestrator** (main repo on `master`): merges worker branches, runs global verification, updates `functions.csv`
- **Workers** (git worktrees on `worker/pool-*` branches): decomp assigned modules, compile, verify locally

### Pool Assignment Protocol
**CRITICAL:** Each pool has a unique `WORKER-pool-{letter}.md` file (e.g., `WORKER-pool-a.md`).
This is the **single source of truth** for that pool's assignment. Pool-specific filenames prevent rebase clobbering.
1. **READ the pool's `WORKER-pool-{letter}.md`** before reassigning — check if it's already doing useful work
2. **WRITE `WORKER-pool-{letter}.md`** in the pool's worktree when assigning new work
3. These files are tracked in git and survive rebases (unique filenames = no conflicts)
4. `data/pool_assignments.md` is just a quick overview — the WORKER file wins if they conflict
5. Pools that ignore their assignment waste hours of compute — this is a serious problem

### Worker Rules
1. **ONLY edit files in your assigned file territory** — see your `WORKER-pool-{letter}.md`
2. **NEVER modify `data/functions.csv`** — the orchestrator handles this exclusively
3. **NEVER edit files assigned to other workers**
4. Commit to your worker branch, never push to master directly

### Worker Code Quality Rules

**ALL WORKER DIFFS ARE REVIEWED BY `tools/review_diff.py` BEFORE MERGE.**
The orchestrator will REJECT diffs that fail the reviewer. Do not submit code that violates these rules — it wastes your compute budget and the orchestrator's time.

1. **Write C++ that a Bandai Namco developer would have written.** Use the struct definitions in `include/app/` and `include/resource/`. If Ghidra shows a pattern, understand it and rewrite it idiomatically — do NOT copy-paste Ghidra pseudocode.
2. **STRUCT FIRST — define types before writing functions.** If a function accesses 3+ distinct offsets on the same pointer, you MUST define or extend a struct in `include/` BEFORE writing the function. Use `unk_0xNN` for unknown fields, typed fields for known ones. Raw hex offset access on untyped `u8*`/`void*`/`u64*` is REJECTED by the reviewer.
   - **GOOD:** `stage->sub_object` (struct field at +0x738)
   - **BAD:** `param_1[0xe7]` or `*(u64*)(sub + 0x790)`
   - **Exception:** trivial 1-2 field leaf functions (e.g., singleton getters) may use raw access.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbergeron42/ssbu-decomp](https://github.com/sbergeron42/ssbu-decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

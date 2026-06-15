---
trigger: always_on
description: PSP static recompiler: Rust analysis/decode/emit pipeline produces C++17 output; C++17 runtime executes it with SDL2 + OpenGL 3.3.
---

# PSPrecomp V2 - Project Instructions

## 1. Project Overview

PSP static recompiler: Rust analysis/decode/emit pipeline produces C++17 output; C++17 runtime executes it with SDL2 + OpenGL 3.3.

**Core value:** Produce correct, compilable C++ that faithfully represents the PSP binary so that when paired with a runtime, the game boots and renders frames.
**Target binary:** Patapon BOOT.BIN (9,713 Ghidra-detected functions, 237 imported NIDs, 804 mid-entry points)

## 2. Architecture and Data Flow

```
psprecomp analyze --ghidra-dir <ghidra-install>/libexec BOOT.BIN -> analysis.json
psprecomp recompile analysis.json --config games/<id>/game.toml -o output -> output/ (C++17: batch_*.cpp, dispatch.cpp, data_sections.cpp, etc.)
cmake -B runtime/build -S runtime && cmake --build runtime/build -> psprecomp_runtime
```

### Rust Crates (`crates/`)

| Crate | Purpose |
|-------|---------|
| `psp-parser` | ELF/PRX parsing, NID database, relocations (goblin 0.9.3) |
| `psp-ir` | MipsOp enum, DecodedFunction, typed IR for all Allegrex + FPU instructions |
| `psp-decoder` | MIPS32 + Allegrex + FPU instruction decoding, delay slot reordering (two-pass) |
| `psp-optimizer` | Peephole passes (all disabled until Phase 7 -- `OptimizerConfig::default()` all false) |
| `psp-emitter` | C++ code generation, batch emission (rayon), dispatch table, CMakeLists |
| `psp-cli` | CLI entry point with `analyze`, `recompile`, `dump` subcommands |

### Key Output Files (`output/`)

`generated/batch_*.cpp` (per-batch translations), `dispatch.cpp` (address-to-function table), `data_sections.cpp` (.data/.rodata/.bss with 0x07FFFFFFU masking), `init_array.cpp`, `mid_entries.cpp`, `syscall_table.cpp` (generated NID import binding table -- the runtime binds HLE handlers by name at its stub addresses, issue #40), `funcs.h` (forward decls), `include/recomp.h` (recomp_context, register aliases, memory macros), `CMakeLists.txt` (globs `batch_*.cpp` only)

### Key Runtime Files (`runtime/`)

`src/main.cpp` (boot sequence, SDL2 event loop), `src/psp_memory.cpp` (128MB rdram), `src/psp_dispatch.cpp` (RECOMP_LOOKUP, miss handler), `src/psp_scheduler.cpp` (cooperative threading), `src/psp_render_queue.cpp` (condvar render protocol), `src/psp_event_loop.cpp` (SDL2 pump), `include/psp_runtime.h`

### Analysis: `analysis/ExtractAnalysis.java` -- Ghidra headless script (function export, xref mid-entries, callback scan)

### Live Memory Inspection

An external live-memory inspection tool can be pointed at either PPSSPP's debugger API (WebSocket; PPSSPP's debugger auto-listens — find the port with `lsof -i -P | grep -i PPSSPP | grep LISTEN`) or this runtime's debug socket to read structs, set breakpoints/watchpoints, and diff memory between the two.

**Runtime debug socket:** The runtime starts a TCP debug socket on port 9999 automatically (see `runtime/src/psp_debug_socket.cpp`). No setup needed — just run the runtime.

## 3. Key Conventions

These decisions are accumulated from 18+ completed plans. Violating them causes real bugs.

### Addresses and Data
- All addresses in analysis.json are hex strings (never u32/u64 in JSON)
- Function size = maxAddress - entry + 1 (not address-count methods)
- PSP memory addresses mask with `0x07FFFFFFU` (128MB address space)
- PRX modules rebase to `PSP_USER_MODULE_BASE` 0x08804000 (`--load-base` overrides); ET_EXEC loads where linked. Canonical image = psp-parser's relocated `segments[]`; Ghidra is byte-gate-verified, never a byte source (DEBUGGING.md #52)

### Runtime Architecture
- `rdram` is separate from `recomp_context` -- passed as separate function parameter for thread-safety
- Memory access via `psp_mem_read`/`psp_mem_write` with `0x07FFFFFFU` mask
- GL calls MUST go through render request queue (never from game threads -- macOS requirement)
- `thread_local PspThread* g_current` -- each OS thread knows its PspThread without shared lookup
- NO game-specific data in runtime core: per-game code lives in `games/<id>/runtime/` (selected by `-DPSPRECOMP_GAME`, default `patapon`), per-game choices in `games/<id>/game.toml`, binary facts in generated headers (`recomp_module.h`, `recomp_game_config.h`). A fix that only works for one title belongs in its game module, never in `runtime/src`

### Emitter/IR
- Mid-entry dispatch: wrapper sets `ctx->entry_point`, parent switch dispatches to label, clears before goto
- Branch/jump IR stores absolute u32 target addresses (not raw offsets)
- Cross-function branches use `RECOMP_LOOKUP` (not goto -- C++ goto cannot cross function boundaries)
- Decode errors produce empty stubs with error comments (not propagate)
- Deduplicated function names use `_ADDR` hex suffix for ODR safety
- JAL fallback uses `FUN_` prefix (Ghidra convention); module start is named "entry" (not FUN_089ACCD0)
- `ctx->f[N]` array notation for float registers (not `ctx->fN.fl`)

### Build System
- CMake glob must be `batch_*.cpp` only (not `generated/*.cpp` -- stale duplicates cause linker errors)
- SDL2 found via pkg-config (not find_package) on macOS
- GLAD2 requires `gladLoadGL((GLADloadfunc)SDL_GL_GetProcAddress)`
- Optimizer passes disabled until Phase 7; PSP OS does not process `.init_array` (game CRT handles it)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wizardengineer/psprecomp](https://github.com/wizardengineer/psprecomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

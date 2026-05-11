---
trigger: always_on
description: Static recompilation of Xbox Burnout 3: Takedown (2004) for Windows 11.
---

# Burnout 3: Takedown - Static Recompilation Project

## Project Overview
Static recompilation of Xbox Burnout 3: Takedown (2004) for Windows 11.
The goal is to translate the original x86 Xbox code into a native Windows executable.

## Key Facts
- **XBE**: `default.xbe` is a retail Xbox executable, XDK 5849, base address 0x00010000
- **Entry Point**: 0x001D2807 (retail, XOR-decoded with 0xA8FC57AB)
- **Engine**: Criterion's custom RenderWare fork (~3.7), statically linked
- **Code Size**: 2.73 MB in .text section (game + CRT + RW engine)
- **Kernel Imports**: 147 Xbox kernel functions to replace with Win32
- **Libraries**: 11 statically linked XDK libs (D3D8LTCG, DSOUND, XMV, XONLINE, etc.)

## Repository Structure
- `docs/` - Detailed analysis and documentation
- `tools/xbe_parser/` - XBE file parser (Python)
- `tools/disasm/` - Disassembly tools (planned)
- `tools/asset_tools/` - Asset conversion (planned)
- `src/kernel/` - Xbox kernel replacement layer (Win32)
- `src/d3d/` - D3D8→D3D11 graphics abstraction
- `src/audio/` - Audio system (DSOUND→XAudio2)
- `src/input/` - Input system (XPP→XInput)
- `src/game/` - Decompiled game code

## Git/GitHub
- Remote: https://github.com/sp00nznet/burnout3.git
- Game assets (`Burnout 3 Takedown/`) are gitignored (large binary files)
- All toolchain code and documentation IS committed

## Conventions
- Python tools use Python 3.10+
- C/C++ code targets MSVC (Visual Studio 2022) or MinGW-w64
- Addresses are always shown as hex with 0x prefix (e.g., 0x001D2807)
- Xbox kernel function names use their original Xbox names with `xbox_` prefix when reimplemented

## Current Work State (Session 43)

### Status: PLAYABLE — driving on real track geometry at 32fps
Game boots to menus, R key launches race, player drives on real Burnout 3 track geometry rendered through recompiled gen code pipeline. Gen code render chain (sub_00351490→sub_00351770_gen→sub_00350C10) active. Gameplay state machine transitions 5→4→5 working. Track geometry drawn via D3D8 DrawPrimitiveUP with pre-transformed vertices. Chase camera follows physics body. Pre-race cinematic plays (flyover camera cuts). 6000 verts/frame, 7 batched draws, 32fps.

### Session 43 Changes (2026-03-20)
- **Gen code render chain un-stubbed**: sub_00351490 (begin camera), sub_00351770_gen (62K scene render), sub_00350C10 (end camera) all running
- **D3D8LTCG device context fully fixed**:
  - GPU read pointer trick: device+0x30 → device+0x2C (eliminates PB spin loops)
  - PB ring management: +0x24/+0x28/+0x2C/+0x30/+0x34/+0x38/+0x44/+0x48 all patched post-snapshot
  - RT surfaces at +0x1974/+0x1978 initialized non-NULL
  - Camera active flag (device+8 bit 14) forced per-frame
- **24 D3D8LTCG stubs fixed**: Proper Xbox stack cleanup calling conventions (esp += 4/8/12)
- **Gameplay state machine**: R key launches race via fe_start_race()
  - State transitions: menu(5) → race init(4) → gameplay(5)
  - Phase state machine (sub_001AA100): phases 1→3→4→5→6→7→9→0x13
  - Phase 9 timeout after 30 tries (render list builder can't complete without RW track loading)
  - g_race_init_done signal prevents state oscillation
- **Track geometry rendering**: Direct D3D8 DrawPrimitiveUP with batched pre-transformed vertices
  - 6000 verts/frame from 49 track chunks, 7 draw calls
  - View-projection transform, distance culling (500 unit radius)
  - Triangle strip → triangle list conversion
- **Chase camera**: Physics body-based fallback with 75° FOV, heading-relative positioning
- **Float ICALL guard**: Rejects IEEE 754 values (exponent 50-200) as vtable entries
- **Input system**: fe_menu_is_racing() fallback for gameplay detection
- **Race launch flow**: R key → fe_start_race → state=4 → phase 1-9 → timeout → state=5 → gameplay
- **g_force_respawn**: Resets physics init flag when launching race (prevents boot-time flag blocking)
- **D3D8LTCG device context reference doc**: `docs/d3d8ltcg_device_context.md` (reusable)
- **Log noise reduced**: Mirror failures (3+50K), SKIP-READ (10+100K)

### Session 42 Changes (2026-03-17)
- **Live render pipeline investigation**: Attempted to enable original sub_001AE6F0 call chain
  - sub_0003FEE0 (RW state setup) crashes on uninitialized D3D device context fields
  - Even with D3D8LTCG mid-entry stubs, gen code walks pointer chains through device+0xCA0+
  - sub_001AD350 (render list dispatch) returns immediately: render entry table res[+24]=NULL
  - Render entry table populated by sub_0003FEE0 chain → circular dependency
- **D3D8LTCG mid-entry stubs added** (recomp_manual.c):
  - sub_0034F5B0 (70K render state flush) — stubbed, returns current PB position
  - sub_003558A0 (45K render state flush) — stubbed, returns current PB position
  - sub_0034D410 (79K render state flush) — stubbed, returns current PB position
  - These are all entry points into the same giant D3D8LTCG function (ends at 0x360A54)
- **D3D device context snapshot captured from xemu** (`src/nv2a/d3d_device_snapshot.h`):
  - Device is STATIC at 0x0035D6A0 (in D3D8LTCG section), NOT heap-allocated
  - 16KB snapshot captured during menu rendering (game state 5, cam=0x4D4008)
  - Loaded at boot via memcpy, with PB/surface/viewport fixups
  - Capture script: `tools/xemu_debug/capture_d3d_device.py`
- **Key findings**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sp00nznet/burnout3](https://github.com/sp00nznet/burnout3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->

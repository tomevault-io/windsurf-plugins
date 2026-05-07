---
trigger: always_on
description: DX9 FFP Proxy porting guide for RTX Remix compatibility. Use when porting a DX9 shader-based game to the fixed-function pipeline.
---


# DX9 FFP Proxy — Game Porting Guide

You are helping a user port a DX9 shader-based game to the fixed-function pipeline. Each game folder under `patches/<GameName>/` is a self-contained remix-comp-proxy project (copied from the template at `rtx_remix_tools/dx/remix-comp-proxy/`). The goal is RTX Remix compatibility: Remix requires FFP geometry to inject path-traced lighting and replaceable assets. Also use the Vibe RE tools (retools, livetools) for static and dynamic analysis to assist with developing this wrapper. They are meant to be used together.

**SKINNING IS OFF BY DEFAULT.** Do NOT enable skinning, modify skinning code, or discuss skinning infrastructure unless the user explicitly asks for character model / bone / skeletal animation support. Until then, treat skinning as non-existent. When the user does request it, read `src/comp/modules/skinning.hpp` and `src/comp/modules/skinning.cpp` for the full implementation.

**SKINNING APPROACH: FFP indexed vertex blending, NOT CPU matrix math.** When skinning is enabled, keep BLENDINDICES and BLENDWEIGHT in the vertex declaration and buffer, upload bone matrices via `SetTransform(D3DTS_WORLDMATRIX(n), &boneMatrix[n])`, enable `D3DRS_INDEXEDVERTEXBLENDENABLE = TRUE`, and set `D3DRS_VERTEXBLEND` to the weight count. CPU-side vertex skinning is a **last resort** -- it is extremely expensive and tanks frame rate. Always prefer the hardware path.

---

## What remix-comp-proxy Does

Each game's remix-comp-proxy folder is a C++20 compatibility mod based on remix-comp-base that intercepts `IDirect3DDevice9` and:

1. Captures vertex shader constants (View, Projection, World matrices) from `SetVertexShaderConstantF`
2. Parses `SetVertexDeclaration` to detect per-element attributes: BLENDWEIGHT+BLENDINDICES (skinned), POSITIONT (screen-space), NORMAL presence, and per-element byte offsets and types
3. Routes `DrawIndexedPrimitive` by vertex layout:
   - No NORMAL -> HUD/UI pass-through (uses different VS constant layout than world geometry)
   - Skinned with skinning module enabled -> FFP indexed vertex blending
   - Rigid 3D (has NORMAL) -> NULLs shaders, applies FFP transforms, draws
4. Routes `DrawPrimitive` by declaration state: world-space draws (have decl, no POSITIONT, not skinned) engage FFP; screen-space and no-decl draws pass through
5. Applies captured matrices via `SetTransform` (FFP)
6. Sets up texture stages and lighting for FFP rendering (stages 1-7 disabled to prevent stale auxiliary textures reaching Remix)
7. Chain-loads RTX Remix (`d3d9_remix.dll`)

## Source File Map

| File | Role |
|------|------|
| `src/comp/main.cpp` | DLL entry, module loading, initialization |
| `src/comp/modules/renderer.cpp` | Draw call routing -- `on_draw_indexed_prim()` and `on_draw_primitive()` |
| `src/comp/modules/renderer.hpp` | Renderer class, `drawcall_mod_context` for save/restore state |
| `src/comp/modules/d3d9ex.cpp` | `IDirect3DDevice9` hook layer -- intercepts all 119 methods |
| `src/comp/modules/d3d9ex.hpp` | D3D9 hook declarations |
| `src/comp/modules/skinning.cpp` | Skinning module (vertex expansion, bone upload, FFP blending) |
| `src/comp/modules/skinning.hpp` | Skinning class interface |
| `src/comp/modules/diagnostics.cpp` | Diagnostic logging to `ffp_proxy.log` |
| `src/comp/modules/diagnostics.hpp` | Diagnostics class interface |
| `src/comp/modules/imgui.cpp` | ImGui debug overlay (F4 toggle) |
| `src/shared/common/ffp_state.cpp` | FFP state tracker -- engage/disengage, matrix transforms, texture stages |
| `src/shared/common/ffp_state.hpp` | `ffp_state` class with all state accessors |
| `src/shared/common/config.cpp` | INI config parser for `remix-comp-proxy.ini` |
| `src/shared/common/config.hpp` | Config structures: `ffp_settings`, `skinning_settings`, etc. |
| `remix-comp-proxy.ini` (in `assets/`) | Runtime config: `[FFP]`, `[Skinning]`, `[Diagnostics]`, `[Remix]`, `[Chain]` |
| `build.bat` | Build script: outputs d3d9.dll proxy |

The codebase is C++20 with a `build.bat` build script, component module system for extensibility.

## What Needs to Change Per Game

The VS constant register layout is defined in `src/shared/common/ffp_state.hpp` as member defaults. Edit these when porting, then rebuild:

```cpp
int vs_reg_view_start_ = 0;    int vs_reg_view_end_ = 4;
int vs_reg_proj_start_ = 4;    int vs_reg_proj_end_ = 8;
int vs_reg_world_start_ = 16;  int vs_reg_world_end_ = 20;
int vs_reg_bone_threshold_ = 20;   // first register treated as bone palette
int vs_regs_per_bone_ = 3;        // 3 = 4x3 packed, 4 = full 4x4
int vs_bone_min_regs_ = 3;        // min count to qualify as bone upload
```

**Bone config:** Run `find_skinning.py` to determine bone start register and upload pattern. Some games upload all bones at once; others upload in groups until hitting a max (e.g., groups of 15, max 75). If grouped, lower `vs_bone_min_regs_`. If bone uploads overlap with non-bone constants, raise `vs_reg_bone_threshold_`.

Beyond the INI config, users may need to modify:
- `renderer.cpp` `on_draw_indexed_prim()` -- draw call routing (which draws get FFP vs shader pass-through)
- `renderer.cpp` `on_draw_primitive()` -- UI/particle handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ekozmaster/Vibe-Reverse-Engineering](https://github.com/Ekozmaster/Vibe-Reverse-Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

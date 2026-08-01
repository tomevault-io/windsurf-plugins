---
trigger: always_on
description: Context file for AI assistants modifying UtilityRelight. Read this before making changes.
---

# CLAUDE.md

Context file for AI assistants modifying UtilityRelight. Read this before making changes.

---

## What this tool is

A Nuke node that relights real footage using AI-estimated utility passes (N, P, Z, ao from NormalCrafter / LiveActionAOV). Outputs a standalone render layer the comp artist merges onto the plate downstream.

- **Two files, both ship to `~/.nuke/`:**
  - `utility_relight.py` — Python module, builds the Group node programmatically
  - `UtilityRelightKernel.blink` — BlinkScript GPU kernel doing the lighting math
- **No `.gizmo` file.** Nuke's `.gizmo` TCL parser is fragile; we build the node via `nuke.nodes.*` calls inside a Python `with group:` block instead.
- Target Nuke version: **16.0**. Probably works on 14–15. Uncertain on 13 — see Known Issues below.

---

## Architecture

```
User-facing Group node
├── Tabs: Channels, Lighting, Key, Spec, Rim, Bounce, Glow, Fog,
│         Occlusion, Output, About
├── Hidden Internal tab: has_p, has_z, has_ao, auto_scaled flags
│
└── Internal DAG (v1.11, always-on 3D preview):
    Input_src ─────────────────────────┐
    Input_aov ─────┬──── Shuffle_N ────┤
                   ├──── Shuffle_P ────┤
                   ├──── Shuffle_Z ────┤
                   ├──── Shuffle_AO ───┤
                   └──── Shuffle_Alb ──┘
                                       └── BlinkScript (RelightKernel) ──┬──► Merge2 in 1 (B, PASSED)
                                                                         │
                                                                         └──► PositionToPoints2[in 0=color]
                                                                             Shuffle_P ──► [in 1=pos]
                                                                             Shuffle_N ──► [in 2=norm]
                                                                             │
                                                                         TransformGeo (X/Y flip)
                                                                             │
                                                                         ScanlineRender (bg=none, result discarded)
                                                                             │
                                                                         Merge2 in 0 (A, IGNORED)
                                                                             │
                                                                         Output1

    LightAxis (Axis2, DAG-orphaned; read by kernel via world_matrix expressions)
```

**Why the DAG looks that way:**
- PositionToPoints2 input 0 is fed from the **kernel output** so the 3D preview shows the RELIT points. Drag the LightAxis, cloud re-lights in realtime. This is the killer feature — don't break it.
- PositionToPoints2 has 3 dedicated inputs (color/pos/norm). Wire each explicitly to the right source. DO NOT try to be clever with shuffled channels on a single input. (I tried. It breaks.)
- LightAxis is not scene-connected. Nuke's 3D viewer picks it up automatically from the group namespace, and the kernel reads it through `world_matrix` expressions. No Scene node needed.
- Single `Output1`. **Nuke groups can only have ONE Output node.** If you see `Output2` getting auto-created, it means two `nuke.nodes.Output(...)` calls are happening — find and kill the duplicate.
- **The Merge2 + ScanlineRender cheat (v1.11):** the shipped 2D output is the kernel's relight. The ScanlineRender branch isn't used for pixels — but its presence upstream of Output keeps Nuke's 3D viewer engaged with the PositionToPoints2 + LightAxis geometry continuously. Press Tab in the viewer any time to drop into 3D mode. No dropdown toggle needed.
- **Merge2 operation=copy** means "B straight through" — B is the kernel, A (ScanlineRender) is ignored at pixel level but attached for the DAG-presence effect.

---

## Kernel design

**Output = sum of 6 independent layers** (key + spec + rim + bounce + glow + fog).

All lit where the light reaches, black everywhere else. It's a render layer, not a final comp.

**Critical math invariants:**
1. **N and P must be in the same Y/Z frame** before dot products. The `nFlipX/Y/Z` + `nSwapYZ` params realign the estimator's N to match P. Default `flipY=1, flipZ=1` is for NormalCrafter-style OpenGL view-space normals paired with image-space Y-down positions.
2. **Softness uses avgNdotLAtten directly**, not divided by avgAtten. The divide flattens the NdotL gradient across the subject. If you "optimize" this away, lighting goes flat.
3. **Rim is direction-aware**, gated by `(1 - NdotL)^wrap`. It rotates with the light. A pure Fresnel(view) rim (which I tried first) is always at the camera silhouette regardless of light position — wrong for a CG-style back-rim.
4. **keyColor always tints the light**, independent of `keyPlateMix`. The plateMix only controls whether the surface color is the plate or neutral white. This is the fix for an earlier bug where the Color slider did nothing at default settings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lettidude/LiveActionAOV](https://github.com/lettidude/LiveActionAOV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

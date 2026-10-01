---
trigger: always_on
description: Attract/3D transform stack — uplifted camera + MAME as reference, not gold standard
---


# Attract / 3D transform stack

## Goal

Original-or-better fidelity for attract (and later race) 3D. **Not** “same as MAME including its limitations.” Decomp exists so lifted game semantics can exceed a broken or incomplete emulator.

## Sources of truth (priority)

1. **Uplifted i960** — camera composition (`comm_attract_geo_script_finish` modes, TGP markers, `copy_catalog` pose).
2. **MAME Model 2 sources as reference only** — operator contracts for GEO (`0x0B` matrix, `0x09` focal, `0x03` window/centers, project formula). Never acceptance criteria.
3. **Decoded point geometry** — catalog/PRG meshes and the Python track viewer for structural checks.

Do **not** use MAME runtime captures (see `no-mame-runtime-capture.mdc`).

## Closed transform stack

| Stage | Source |
|-------|--------|
| View | Uplifted attract/TGP composition → current matrix |
| Upload | TGP `0x05` → GEO `0x0B` + 12 floats (`copy_catalog` `+0x34`) |
| Object pose | `copy_catalog` push / translate / rotate / draw / pop |
| Project | GEO `0x09` + `0x03`; MAME `model2_3d_project` as formula reference |

Apply **only** transforms the lifted code and GEO commands issue. No invented FOV, look-at, or mesh auto-frame.

## Viewer role

OpenGL (or any host display) is a **transform debugger**, not a second renderer. Prefer:

- Modelview: identity when verts are already view-space (post-`0x0B` object draws)
- Projection: latched focal + window only

Defer a full rasterizer port until this stack is trusted.

## Current work order

1. Camera modes from disasm (`script_finish` jump table @ `0x11C20`), starting with modes 7 and 4
   — `script_finish` locals (`0x40(fp)`…`0x88(fp)`) must be a **private host frame**
     (stack buffer), not `fp = sp`. Attract `bind_fp` uses a shadow buffer while
     `script_frame_setup` was leaking `sp += 0x30`; tying locals to `sp` segfaulted.
     `ld (g0)` still reads the caller’s string_draw object; scene stays in the
     private frame.
   — `geo_attract_copro_vec_scale` is a real `call` (`0x123B0`→`0x11A80`): give it a
     **private host frame**. Unit temps at callee `0x40(fp)` must not share the
     caller's scene; `a2` (pre-call `lda 0x40(fp)`) keeps the scene pointer.
     Shared global `fp` was a lift bug (wrong eye.xy; eye.z overwritten by unit.z).
   — **Modes 4 never rotate R in script_finish.** Mode 7/8 `vec_scale` issues
     TGP `0x54` then `0x2f` then `0x27` with **no** following `0x25`. Race
     `geo_view_scene_frame` does `0x2f` then explicit `0x25` (normalize
     readback only). Host `0x2f` is **normalize-only**; look-along R comes from
     `0x54` (mode 7 keeps that R). Bare race `0x2f` (table_index_b, force_slot)
     must not orient — that invented side effect spun chase after road hits.
   — Mode 4: `0x54(scene)` then `T = −(obj−scene)`. `0x54` orients R along
     scene so cars at cam≈scene sit on +Z (close-up). Focus `@ 0x20220c` stores
     the delta (pen yaw), not a look target.
   — Mode 7: dolly `eye = scene − scale·normalize(obj−scene)` with
     `scale = radius − |delta|·falloff` from float pair at `lda 0x5acbe0[idx]`.
     `0x54(delta)` look-along, `0x2f` unit readback, then `0x27`.
   — Modes 2/8/9/11 read the **pair** object (`g1` / `pair_obj` from
     `0xa0(fp)`), not `g0`. Mode 7/1/4/5/10 use `g0`. Banner calls pass
     `g1=0` — those modes must not run (or no-op). Mode 7/8 share
     `vec_scale` @ `0x11A80` with private callee frame + restored `sp`.
   — Modes 5/6: `geo_attract_fp_series` @ `0x33C00` is cubic **B-spline**
     (N_i,3), not Bezier. At `t=0` cam ≠ table P0; for mode-6 table
     `@ 0x5ae400` smoke is `≈(-114, 38, -553)`. Load table via `i960_ld`
     (ROM mirror); `stl`/`ldl` at `0x50(fp)` are full g8:g9 pairs; private
     callee frame. Mode 6: `t=link/412`, endpoint immediates, `0x54(end−cam)`
     then `0x27(−cam)`. Mode 5: indexed base `0x5ae3d0+…`.
   — TGP `0x54`: store direction **and** set R to look along the unit (Y-up).
     `0x2f`: normalize readback only. Race `geo_view_scene_frame` clears with
     `0x25` after `0x2f` when it only wants the unit. Attract modes that
     translate after `0x54` without `0x25` keep that R — not an invented look-at.
   — Race `road_span_basis` @ `0x31320`: nested `0x25`/`0x27(bx,0,cx)` then
     TGP `0x51(bz,0,by)` orients R (fw `@ 0x48E`) before `0x2c(ox)` →
     `node+0x90/98` world XZ. Host must HLE `0x51` (look-along family as
     `0x54`); no-op left local snaps that pitch-wind chase. `0x60` stores
     matrix only (no R effect).
2. Keep TGP HLE tied to uplifted call sites
3. Validate mesh against Python track viewer; if points match and camera differs, fix uplift/TGP/GEO — not the rasterizer
4. Attract inner loop: 0→9 lifted (`inner_8` @ `0xF7D0` resets `0x20209c`;
   `inner_9` @ `0x13810` → `comm_attract_board_tick` @ `0x16540`, flags/scene
   tick — not on cam path). After `vec_scale` private-frame fix: re-check mode
   7/8 framing; if still off, audit object pose / GEO project, not invented look-at.
   — `geo_draw_frame_entry` @ `0x4E90`: GEO `0x09` is `stl (450,450)` (both lanes);
     attract path uses `geo_fifo_bootstrap` focal **350** directly — correct for
     first shot. Feeder early path must `ldq`/`stq` catalog (word3 → `0x20b940`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xandoxan65/segarally95-recomp](https://github.com/xandoxan65/segarally95-recomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

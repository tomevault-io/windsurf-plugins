---
trigger: always_on
description: A matching decompilation of the US PS1 release of Vandal Hearts (`SLUS_004.47`), carried through
---

# Vandal Hearts Decompilation → Native PC Port — Project Context

## What this is

A matching decompilation of the US PS1 release of Vandal Hearts (`SLUS_004.47`), carried through
to a **working, packaged, cross-platform native PC port**. The decomp uses the standard PSX-decomp
toolchain: [splat](https://github.com/ethteck/splat) for disassembly/extraction,
[maspsx](https://github.com/mkst/maspsx) to post-process GCC 2.x asm for the PSX assembler quirks,
an old GCC 2.x frontend (`cc1_v263`/`cc1_v257`, prebuilt by
[decompals/old-gcc](https://github.com/decompals/old-gcc)) for compilation, and a
`mips-suse-linux-*` binutils cross-toolchain for assembling/linking. The match is verified by
`md5sum`-comparing the rebuilt `SLUS_004.47` against the original.

## Project stages — read this before making architectural calls

The objective was always a **native PC port**, not just a matching rebuild. Two foundational stages
are **complete**; a third (gameplay/QoL) is **underway** (first release shipped).

1. **Stage 1 — matching decomp (✅ done 2026-07-10).** The C source is bit-identical to the
   original: `make check` produces a byte-exact `SLUS_004.47` (`md5 596bb082a2de5f1fe977dd3d7e160b03`,
   confirmed by both `md5sum` and `cmp`). This is the foundation, and it is **still enforced on
   every change** — see *Byte-exact discipline* below.
2. **Stage 2 — de-consolization / native PC port (✅ complete 2026-07-24).** Every PSX hardware
   interface — GPU packet submission (`libgpu`), GTE matrix math (`libgte`), CD-ROM/XA audio
   (`libcd`/`libpress`), SPU, MDEC video, pad input — is replaced with a portable equivalent
   (SDL2 + OpenGL + OpenAL, chosen after evaluating Vulkan). The **full game runs end-to-end** from
   the real disc, validated by full playthroughs on **Windows and Linux** including the endgame and
   credits. Details below and in [`docs/`](docs/).
3. **Stage 3 — gameplay/QoL (v1.1–v1.6 shipped; main additions complete, now maintenance).** PC-only
   quality-of-life, balance and graphics work, all `platform/pc/` or `#ifdef PC_FEAT`-gated. **v1.1.0 released 2026-07-25** (bidirectional
   shoulder-button ally-cycle; enemy threat overlay; a **SELECT+START** in-game options overlay,
   right-stick axis invert — Y inverted by default — plus deploy-relative saves). **v1.2.0 released
   2026-07-25** (video: window scale/fullscreen in the overlay; **save management** — unlimited
   whole-card backups via `saves/.archive/`, restore/delete with a "back up first" safe default, a
   `(*)` active-card marker, and a Start-to-inspect detail view of each backup's 3 slots). The overlay
   is now a small menu system (`platform/pc/src/pc_overlay.c` + `pc_saves.c`; screens MAIN/SAVES/
   CONFIRM/DETAIL). **v1.3.0 + v1.3.1 released** — **Tactical Mode**, a large opt-in rebalance
   (per-chapter level cap, Trials that reward gold + XP, class reworks, a reined-in Vandalier, clarified
   item text), validated across a full playthrough; normal mode stays byte-for-byte retail. Player guide
   [`docs/tactical-mode.md`](docs/tactical-mode.md), [`docs/known_issues.md`](docs/known_issues.md),
   roadmap [`docs/roadmap.md`](docs/roadmap.md). Note: the "zero `src/` edits" ideal was the
   *balance-package* rule — other Stage-3 features need gated `src/` hooks. **v1.4.0 released 2026-07-31**
   — a QoL pass: battle fast-forward (2×), controller-aware overlay labels (Xbox/PlayStation), magic-
   resistance-aware Tactical AI, finer camera elevation (incl. keyboard R/F). **v1.5.0 released** — a
   higher-fidelity graphics track. Three pieces: (1) the **PSX-accurate software rasterizer**
   (`VH_ACCURATE`, default-on) — a fixed-point integer DDA matching the PS1 GPU's exact coverage + texture
   UVs, ordered dithering (gated on the GPU dither bit), 5-bit blend, ~99.8–99.99% pixel-exact vs a
   DuckStation VRAM capture (legacy renderer kept as `VH_ACCURATE=0`, an INI-only softer alternate); (2)
   **internal-resolution supersampling** (`VH_INTERNAL_SCALE` 1–4×, off by default) layered on that DDA —
   sharper 3D with no re-authored assets, with built-in *crust-free* tile sampling (biases the finer hi-res
   sample onto tile interiors like the reference renderer, so no dark tile-seam grid; 2D UI/text auto-kept
   pixel-aligned); (3) a **multithreaded rasterizer** (`VH_RASTER_THREADS`, auto) so 4× holds 30 fps and
   battle fast-forward stays effective. The tile-seam grid, compass "dotted lines" and parked water-shimmer
   were all resolved by crust-free sampling. Guiding principle: *sharpen without reinterpreting* (no
   upscaled video, no redrawn sprites, no camera-feel changes). **v1.6.0 released 2026-08-02** — an
   **optional HD pack** (the base build ships no art and is unchanged without one). Upscaled `.webp`
   backgrounds replace the 320×240 pre-rendered art via a content hash of each `LoadImage` VRAM upload
   (sampled in the 1.5 hi-res pass); the intro/ending FMVs can be swapped for HD **H.264/HEVC** re-encodes
   (a `libav` decoder in `platform/pc/src/pc_hdvideo.c`) while the game keeps its original XA audio + frame
   timing. New deps **libwebp** + **libav** are default-on (`NO_WEBP`/`NO_HDVIDEO` opt-outs); the Windows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HalmyLyseas/VandalHearts-PcPort](https://github.com/HalmyLyseas/VandalHearts-PcPort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->

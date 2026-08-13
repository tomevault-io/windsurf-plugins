---
trigger: always_on
description: Repo-wide notes for AI coding agents working on this codebase. This file is
---

# AGENTS.md

Repo-wide notes for AI coding agents working on this codebase. This file is
**durable guidance**: how to measure, what the invariants are, and what not to
do. It should stay roughly this size.

Dated investigations — what was tried on a given day, with the before/after
tables and the reasoning — live in [LOG.md](LOG.md). Put new findings there and
leave only the standing rule here. If a LOG entry has no consequence for future
work, it does not need a line in this file.

## Standing rules

**Verify assembly against C, and prove the check can fail.** `renderer_pack.c`'s
`compare_stride2_column_asm` (DEBUG_PERF) runs the asm span and the per-tile C
reference into two canary-guarded scratch blocks and compares. Read back
`asm_mismatches`, `asm_canary_failures` and `asm_checked_tiles` after any change
to `src/renderer/renderer_hotpath.s`. If you touch the harness itself, also inject a
deliberate fault and confirm it reports non-zero before reverting — this harness
spent months silently comparing asm against asm and reporting success (LOG,
2026-08-04).

**Not every route is valid for an A/B comparison.** Only routes that end at the
same player pose in both builds can be compared by run average; a faster build
walks somewhere else and rasterizes a different scene. `stationary-combat`,
`checkpoints` and `slow-turn` have held pose-stable; `tour-east-combat` and
`barrel-pointblank` do not. Confirm by checking that the mixed/flat tile counts
(or billboard byte counts) are identical on both sides before believing a delta.

**The view tilemap is column-major**, `view_tile_index(tile_x, tile_y) =
tile_x * VIEW_TILE_H + tile_y`. A column's tiles are contiguous, and screen row
`y` of byte lane `L` sits at `(y>>3)*32 + (y&7)*4 + L`, which is identically
`4*y + L`. Two shipped optimizations rest on this, and one real bug came from
assuming row-major (LOG, 2026-08-03).

**Budgets: 64 KB work RAM, 4 MB ROM.** The work-RAM guardrail is the binding
one — `tools/check-rom.ps1` errors below 16 KB free and SGDK panics at boot
around 13.7 KB. ROM is *not* tight: `.text` is ~1.39 MB against a 4 MB cap, so
there is ~2.6 MB of headroom. The 1408 KB figure some notes used is just where
`sizebnd` pads `out/rom.bin`, not a limit. Check `size.exe out/rom.out` against
the 4 MB cap before ever calling a precompute-vs-compute tradeoff unaffordable.
`ENABLE_BANK_SWITCH` (SSF mapper, 12 MB) exists but its `0x300000` window is for
cold bulk assets — never put a table sampled at pixel rate behind it.

**Fidelity tradeoffs need the user judging motion.** A static screenshot
approval does not survive real gameplay; the stride-4 revert (LOG, 2026-07-27)
is the precedent.

## Dead ends — do not redo without new evidence

Each is measured and written up in [LOG.md](LOG.md); the date locates the entry.

- **Partial base-bank upload** — the 300-tile upload is ~2 vblanks of a ~12
  vblank frame; DMA-side levers cannot recover what the CPU is spending
  (2026-07-19).
- **`RAY_COL_STRIDE` 4** — shipped, then reverted. Faster, but the user rejected
  the fidelity loss in motion (2026-07-22 → 2026-07-27).
- **Billboard projection spatial pre-cull** (2026-07-21) and **visible-subsector
  billboard cull** (2026-07-29) — both cost more than they saved.
- **Normalizing `fx_sin`/`fx_cos` to unit amplitude** — renders the world ~20%
  larger and costs 2.7x in sprite rasterization. See the invariant below
  (2026-07-30).
- **Duplicating `FREEDOOM_WALL_PACKED_PAIRS` to drop `andi.w #63`** — it fits in
  ROM now, but the post loops are only ~a quarter of the pack stage, so it would
  buy ~1% for +736 KB (2026-08-04).
- **Applying the billboard gather/apply row cache unconditionally** — +44% on
  `stationary-combat`. It is gated on magnification for a reason (2026-08-04).

## How to measure renderer frame cost (headless, no BlastEm UI needed)

1. Build with perf instrumentation and the deterministic-route checkpoint hook:
   ```
   $env:EXTRA_FLAGS="-DDEBUG_BLASTEM_CHECKPOINT=1"; .\tools\build-windows.ps1 -DebugPerf
   ```
2. Resolve the perf mailbox symbol:
   ```
   python tools/resolve-symbol.py out/symbol.txt g_debug_perf_mailbox --bytes 128
   ```
3. Run the deterministic route through the custom BlastEm build and capture a report:
   ```
   .externals/blastem/build/windows/blastem.exe -b 600 out/rom.bin \
     --md-route tools/routes/checkpoints.txt --md-report out/perf.json \
     --md-perf-mailbox <ADDR>:128
   ```
   `-b <frames>` is required or BlastEm never exits and no report is written.
4. Decode the `perfMailbox` hex blob as `RendererPerfSnapshot` (see
   `src/renderer/renderer_perf.h`) using **m68000 struct alignment** (u16/u32 align to
   2 bytes, bool is 1 byte) — a plain-C decode gets the field offsets wrong.

`tools/routes/checkpoints.txt` ends in sustained movement (no combat/rotation
segment) — it is the **worst case for temporal-coherence-based optimizations**.

`tools/routes/tour-east-combat.txt` (7335 frames, run with `-b 7400`) is the
first route that actually leaves the start room: through the NE corridor, C-opens
the group-0 door at x=1536, fights past enemies into the zigzag area, ending at
(2416, 3081) angle 106 (verify via the `g_player` mailbox). Its scenes are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [celsowm/megaldoom](https://github.com/celsowm/megaldoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

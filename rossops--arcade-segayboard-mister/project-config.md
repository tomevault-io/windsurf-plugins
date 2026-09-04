---
trigger: always_on
description: This repo started on 2026-08-27 as a copy of the reusable half of the X Board
---

# Sega Y Board core — working notes for Claude

This repo started on 2026-08-27 as a copy of the reusable half of the X Board
core, `/Volumes/roms/Arcade-SegaXBoard_MiSTer` (GitHub rossops/Arcade-SegaXBoard_MiSTer,
also MiSTer-devel/Arcade-SegaXBoard_MiSTer). Read that repo's `docs/DESIGN.md`
and `README.md` before designing anything: it is the worked example of every
convention below, and its git history shows what each decision cost.

## What is here and what is not
- `docs/references.md` lists every carried-over file. `yb_` used to be `xb_`.
- M0 (branch `m0-scaffold`) trimmed the scaffold: `rtl/yb_pkg.sv`,
  `Arcade-SegaYBoard.sv`, `verif/board/tb_board.sv`, the ROM loader and the
  tools carry the Y Board map, stream and descriptor; `rtl/yb_core.sv` is a
  stub with the final port list and a gradient on the real video timing;
  `tools/romsets.py` has `gforce2`. M1 (branch `m1-cpus`) filled `yb_core`
  with the three 68000s, every CPU-side RAM, the shared-RAM arbiter, the
  math chips, `rtl/io/yb_315_5296.sv` and `rtl/io/yb_msm6253.sv` (golden
  models in `verif/models/`), the scanline IRQs, watchdog and sound latch;
  `tools/mame_trace.py` traces three CPUs and `verif/board/check_m1.sh` is
  the gate. M2 (branch `m2-ysprites`) added `rtl/video/yb_ysprite_5305.sv`
  rendering into the DDR3 framebuffers (`yb_fb_if`, now 512-line buffers)
  with a translation-only scan-out through the palette; golden model
  `verif/models/ysprite5305.py` (render + full and translate-only scanout),
  standalone harness `verif/unit/ysprite/`, MAME dumps via
  `tools/mame_capture.py` into `verif/golden/gforce2/f<N>/`, board check
  `tools/board_check.py`, gate `verif/board/check_m2.sh`. M3 (branch
  `m3-rotation`) added `rtl/video/yb_rotate_5306.sv` (affine scan-out with
  a 128-word DDR3 cache, `verif/unit/rotate/` harness, `verif/models/
  rotate5306.py`, gate `verif/board/check_m3.sh`). M4 (branch `m4-bsprites`)
  added `rtl/video/yb_bsprite_5196.sv` (line-based, private list copy) and
  the 315-5312 mixer in `yb_core`; models `bsprite5196.py`, `mixer5312.py`;
  `tools/frame_check.py` (models vs MAME screenshots), `tools/frame_diff.py`
  (RTL vs MAME screenshots), harness `verif/unit/bsprite/`, gate
  `verif/board/check_m4.sh`. M5 (branch `m5-sound`) wired `yb_soundsys`
  (Z80, YM2151, 315-5218 with the Y Board banking: shift 13, mask F8) to
  the latch NMI, `/SRES` and `/MUTE`; gate `verif/board/check_m5.sh` (PCM
  vs `segapcm.py`, WAV envelope vs MAME's `mame_coin30.wav`). M6 (branch `m6-bringup`) is the
  bring-up: the simulation half is `verif/board/check_m6.sh` (test menu
  pixel-exact vs MAME with the switch pressed at frame 200, the Scene
  Select exact against the model from the board's own dumps, the 16B
  static text shared with MAME); the hardware half (DIPs on the test
  menu, NVRAM across a power cycle, 30 minutes of attract, HDMI vs sim)
  is the user's. Bench dump timing is per layer, see the `+dumpframe`
  comment in `tb_board.sv`. M7 (branch `m7-games`) added the other
  five games: `tools/romsets.py` holds all 19 sets (clones as
  alternatives), analog modes 1-4 with the Power Drift gear toggle and the
  Rail Chase gun cursor in `yb_core`, four MRA button layouts in the top,
  the shared-RAM arbiter's hold across a CPU's read-modify-write cycle
  (Power Drift's tas lock deadlocked without it), `mame_capture.py
  -snapview native` (MAME composes layout artwork into screenshots
  otherwise) and `frame_diff --step-ok --max-far N`; gate
  `verif/board/check_m7.sh`. The bench's `+watch_a/+watch_b` (shared RAM)
  and `+watch_x` (sub X backup RAM) log accesses for chasing CPU
  handshakes, `+hold`/`+start2..5` drive inputs, `ROMWR` logs writes into
  ROM space (acknowledged and dropped since R360 stalled on one), and
  `mame_trace.py --coin/--starts/--cfgdir` traces MAME past inputs. Open:
  the Python model chain is wrong on Power Drift (OQ12) and the link
  version of Power Drift (`pdriftl`) is not supported.
- `sys/` is MiSTer-devel's Template, byte for byte. Never edit it; update it by
  copying the template again. Keep `.qsf` deviations from Template.qsf to the
  handful that are listed in a comment at the top of the file.

## How the work goes
- The plan is `docs/DESIGN.md`: hardware reference from MAME, memory
  placement, module list, milestones M0..M7 with a pass criterion and a gate
  script each (`verif/board/check_mN.sh`), and the open questions. Start at
  M0. Update the README status table as milestones close.
- Every custom chip gets a Python golden model ported from MAME, a cocotb unit
  test, and a place in the Verilator board bench that dumps frames to diff
  against MAME captures (`tools/mame_capture.py`, `tools/frame_diff.py`).
  Unit benches are not enough for board-level sequencing: an X Board CDC bug
  only showed in the board check.
- The SMB share's drive spins down during long simulations and has killed a
  bench run. `tools/keepalive.sh` writes and deletes `DELETE_ME` on the share
  every 30 s; `make -C verif/board run` starts it for the duration of every
  run (so the gate scripts are covered). Any other long job on the share
  (a MAME batch, a Quartus build watched from here) starts it too.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rossops/Arcade-SegaYBoard_MiSTer](https://github.com/rossops/Arcade-SegaYBoard_MiSTer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->

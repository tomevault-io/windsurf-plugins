---
trigger: always_on
description: This repo started on 2026-08-31 as a copy of the reusable half of the Y Board
---

# Sega Space Harrier / Hang-On core — working notes for Claude

This repo started on 2026-08-31 as a copy of the reusable half of the Y Board
core, `/Volumes/roms/Arcade-SegaYBoard_MiSTer` (rossops/Arcade-SegaYBoard_MiSTer,
commit 1ceb568), which itself grew out of the X Board core
(`/Volumes/roms/Arcade-SegaXBoard_MiSTer`). Read both repos' `docs/DESIGN.md`
and `README.md` before designing anything: they are the worked examples of
every convention below, and their git history shows what each decision cost.

## What is here and what is not
- `docs/references.md` lists every carried-over file. `sh_` used to be `yb_`
  (and `sh_adc0804.sv` used to be the X Board's `xb_adc0804.sv`).
- Nothing board-specific exists yet. M0 trims the scaffold: `sh_pkg.sv`
  rewritten for this board's memory map, stream and descriptor; the emu top
  cut down to an `sh_core` stub on the real video timing; `tools/romsets.py`
  filled with `hangon`; the carried Y-specific tools that were dropped
  (board_check, frame_check) rebuilt as the video milestones need them.
- New work this board needs that neither parent had: the 315-5011/5012
  tilemap generator (the older HANGON variant of the System 16 tilemaps —
  the X Board's is the later variant, start from it), the segaic16 road in
  its HANGON/SHARRIER form (the X Board has the later road; adapt, do not
  rewrite), the Hang-On and Space Harrier line-based sprite generators
  (`sega16sp.cpp`: SEGA_HANGON_SPRITES, SEGA_SHARRIER_SPRITES — the Y
  Board's 16B renderer is the closest shape), a YM2203 (jotego's jt03 is
  the candidate, same licence situation as jt51), an i8255 PPI, an MCS-51
  core for Space Harrier's dumped 8751 (315-5163A), and FD1089B table
  decryption for Enduro Racer. The FD1094 (Super Hang-On conversion) can
  wait: `shangonrb` is the unencrypted bootleg.
- `sys/` is MiSTer-devel's Template, byte for byte. Never edit it; update it
  by copying the template again. Keep `.qsf` deviations from Template.qsf to
  the handful listed in a comment at the top of the file.

## The board (from MAME 0.289 `segahang.cpp`, verified 2026-08-28)
- Two 68000s: 25.174 MHz/4 (6.29 MHz) on Hang-On, 10 MHz on Space Harrier
  and Enduro Racer (`sharrier` config replaces the main CPU). Z80 at
  8 MHz/2. Screen 320x224.
- Three sound boards (corrected 2026-08-30 against the driver): YM2203 at
  4 MHz plus 315-5218 PCM at 8 MHz on `hangon`, `sharrier` and `enduror1`;
  YM2151 at 4 MHz plus PCM at 4 MHz on the YM2151 Enduro sets and the
  Super Hang-On conversions; two YM2203s plus PCM at 4 MHz only on the
  `endurob2` bootleg. The PCM banking is the older 512K style, a
  parameter our `sh_segapcm_5218.sv` already has.
- Inputs through an ADC0804 (25.174 MHz/4/6) — carried from the X Board —
  and an i8255.
- Space Harrier's i8751 (8 MHz, ROM 315-5163A, dumped, in MAME's set): its
  data space is a window onto the main 68000's bus (address = port1 bits
  6,5,4,3 as A20,A18,A17,A16 | its 16-bit address, with A0 inverted);
  port 1 bits 2:0 drive the main CPU's IPL; it gets IRQ0 from vblank; it
  writes 0x1c to 150000 in its vblank handler and reads 170000, both
  ignored by MAME; MAME suppresses its write to 40385 as a sync hack; so do we,
  after measuring the race (docs/DESIGN.md M7: the MCU's reset zeros land
  110 ms after the 68000's one-shot heartbeat, on our side and MAME's).
- Encryption: `enduror` FD1089B 317-0013A on the main CPU (table decrypt,
  keys in MAME); `shangonro`/`shangonho` FD1094 317-0038/0039 on the sub.
- Sets: hangon (+hangon1, hangon2 ride-on), sharrier (+sharrier1), enduror
  (+endurora/b, enduror1, bootlegs incl. decrypted endurord/enduror1d),
  shangonro/shangonho (+shangonrb, shangonrb2 bootlegs).

## How the work goes
- The plan is `docs/DESIGN.md`: hardware reference from MAME, memory
  placement, module list, milestones with a pass criterion and a gate
  script each (`verif/board/check_mN.sh`), open questions. Start at M0.
  Update the README status table as milestones close, and keep the
  README's "Core vs MAME" section current: every place the core matches
  MAME exactly, deliberately differs toward the hardware, or where a bug
  showed which of the two was right, gets an entry when it happens (the
  user's standing request, 2026-08-31).
- Every custom chip gets a Python golden model ported from MAME, a cocotb
  unit test, and a place in the Verilator board bench that dumps frames to
  diff against MAME captures. Unit benches are not enough for board-level
  sequencing; two X Board and two Y Board bugs only showed at board level.
- Dump timing is per consumer, not one fixed line: dump each RAM at the
  moment the renderer actually reads it (the Y Board's M6 finding; its
  `tb_board.sv` +dumpframe comment is the worked example).
- The SMB share's drive spins down during long simulations. `tools/keepalive.sh`
  writes and deletes `DELETE_ME` on the share every 30 s; `make -C verif/board
  run` starts it for every run, and any other long job on the share (a MAME
  batch, a Quartus build watched from here) starts it too. The share also
  drops sessions outright ("Operation not permitted" on everything): stop,
  tell the user to kick it, retry.
- Simulation runs on the Mac (Verilator 5, Icarus, cocotb in `verif/.venv`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rossops/Arcade-SegaSpaceHarrier_MiSTer](https://github.com/rossops/Arcade-SegaSpaceHarrier_MiSTer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->

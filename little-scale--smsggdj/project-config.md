---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

SMSGGDJ — an LSDJ-inspired music tracker for the Sega Master System and Game Gear (one tree, two ROM flavors: SMSGGDJ and GGDJ), written in pure Z80 assembly (WLA-DX). Sound is the SN76489 PSG only, including 4-bit PCM and wavetable synthesis via the tone-period-1 DC-DAC trick on channel T3.

**DESIGN.md is the contract.** It records hardware constraints, the data model, and decisions that were already debated (PAL default, CH3-steal policy, control scheme, 24 PPQN sync default). Read the relevant section before making design decisions; don't re-litigate settled ones. SAVEFORMAT.md documents the save/SRAM format and must be kept in sync with any RAM-layout change.

## Build and run

```sh
make          # both flavors: build/smsggdj.sms + build/smsggdj.gg
make run      # launch the SMS ROM in Emulicious (bundled in tools/emulicious/)
make clean
```

Build flags (passed to `wla-z80` via the Makefile): `TARGET_GG` selects the
Game Gear flavor (below). The build boots to a blank song; the baked-in demo
song is loadable from the PROJECT screen (DEMO, a two-press confirm —
`song_init` copies the ROM `demo_song_block`). The boot splash lives in a
bank-1 `"Splash"` section (bank 0/slot 0 is full).

Two ROM flavors from one tree: `TARGET_GG` (assembled as `main-gg.o`) is the
native Game Gear build — 20×18-tile window layout via a UI origin in
`nt_addr_hl` plus per-flavor defines (`GRID_ROW`, `NAME_ROW`, `STATE_*`),
START instead of PAUSE, 12-bit CRAM, NTSC-only, paged WAVE screen. Layout
literals must respect the GG window: columns ≥20 and grid rows beyond 15
don't exist there (DESIGN.md §15).

- Toolchain: `wla-z80` + `wlalink` (Homebrew). `make run` needs `/opt/homebrew/opt/openjdk/bin/java`.
- Emulicious must have `AudioSync=true` in `tools/emulicious/Emulicious.ini`, or it free-runs at turbo speed.
- There is no test suite; verification = build clean and run in Emulicious. Emulicious's PSG-DAC emulation is decent; the timing-critical sample/wave feed **and 32 KB SRAM (6 slots, second bank persists)** are **confirmed on real hardware** (Master Everdrive X7 on a PAL SMS1) — still worth checking NTSC and Game Gear hardware.
- Build-generated includes (made automatically by `make` from the Python tools): `build/font.bin` (makefont.py), `build/notes.inc` (maketables.py — PAL+NTSC note-period tables), `build/demo.bin` (makedemo.py), `build/logo.bin`/`logo.inc` (makelogo.py from `art/`), and the sample pool (see below).
- **Demo song:** if `songs/demo.smdj` exists (a committed save export), the build strips its 16-byte header and bakes the 5376-byte block as `build/demo.bin`, with its echo settings (the SMDJ3 reserved bytes) in `build/demo_echo.bin`; both ride into `song_init`. Otherwise `makedemo.py` composes one. Delete `songs/demo.smdj` for the procedural demo.
- **Sample pool:** if `samples/pool.bin` exists (a 96 KB pool image, the production bank — committed, tuned in `tools/patcher.html`), the build bakes it in verbatim via `smsggdj_sample.py --pool-in`. Otherwise it converts `samples/*.wav` with `smsggdj_sample.py`. Delete `samples/pool.bin` to go back to the WAV pipeline. The pool region is byte-identical in both flavors, so one pool serves `.sms` and `.gg`.
- `tools/savetool.py build/smsggdj.sav list|export|import` manipulates emulator/Everdrive save images (see SAVEFORMAT.md).

## Architecture

Single translation unit: the Makefile assembles only `src/main.asm`, which `.INCLUDE`s every other source file plus the generated includes. There is no per-file linking — adding a file means adding an `.INCLUDE` in main.asm *and* a prerequisite in the Makefile. 128 KB ROM, 8×16 KB banks (standard Sega mapper, `.SMSTAG` header): code/tables/demo in banks 0–1, the self-describing sample pool in banks 2–7 (contract in DESIGN.md §10.3 — ROM file offset $8000). SRAM maps over the pool banks in slot 2, so anything enabling SRAM must `smp_abort` first.

- `src/main.asm` — memory map, hardware port defines, boot, VBlank IRQ dispatch, main loop.
- `src/vdp.asm` — Mode 4 text UI helpers; register init table (line interrupt every 2 scanlines for the sample feed).
- `src/input.asm` — pad read with edge detection and LSDJ-style DAS key repeat.
- `src/psg.asm` — shadow registers in RAM; `psg_flush` writes only what changed. Volumes are stored as *attenuation* (0 = loud, $F = silent).
- `src/engine.asm` — per-tick sequencer pipeline (groove → row advance → trigger/commands → envelope → kill → PSG shadows → mute gate). Channel state = 4 × 32-byte structs walked with IX; layout documented in the file header. Full command set (A/B/C/D/E/F/G/H/I/K/L/M/N/O/P/R/S/T/V/W; `cmd_chars`/`cmd_order` map ids↔letters↔alphabetical rank) runs through one executor shared by phrase and table columns. `B` (wave-bank select) and D/L/I are peeked *before* the trigger; tempo (T)/groove (G)/hop (H)/wait (W) are handled inline. A once-per-tick `echo_pass` post-pass (bank-1 `Echo` section) reads a 64-tick ring of T1 and replays delayed/transposed copies onto T2/T3.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [little-scale/smsggdj](https://github.com/little-scale/smsggdj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

---
trigger: always_on
description: transforms in both builds). All identical.
---

# Galaga for the Amiga 500

A port of the arcade game Galaga to the stock Amiga 500. The bar is
high-quality play-feel and sprites that match the arcade, not a
pixel-perfect copy. Where the Amiga can do better than the original
(the starfield), it should.

This file records what has been decided and why. Update it when a
decision is made or changed.

## Hard rules

- **Bring your own ROM.** The copyright status of Galaga is unclear, so
  nothing from the ROM is committed or distributed: not the ROM
  (`original/`), not anything generated from it (every `build/`
  directory), not the third-party disassembly (`reference/hackbar-galaga/`).
  Tools read the user's own `galaga.zip` (Namco rev. B) at build time.
  Small constants typed into tools (colour values, ROM addresses,
  checksums) are fine.
- **Target machine:** stock A500: 68000 at 7 MHz, OCS, 512K chip RAM,
  PAL, Kickstart 1.3. Do not rely on slow RAM or fast RAM.
- **PAL only** (the user's decision, 2026-10-02). On an NTSC A500 the
  game runs a fifth too fast (72 arcade frames a second) and the bottom
  of the playfield, with the fighter, is off the screen; nothing is
  done about it, and the release says so.
- **Assembler:** vasm (`vasmm68k_mot`), Motorola syntax. Keep the code
  tight. The game's code follows the style guide: @docs/style.md
  (the experiments predate it and do not).
- **Measure, don't estimate.** Timing claims come from the emulator as a
  stock A500 with its cycle-exact emulation on: `game/tools/run_tests.py`
  (see Tests), or `tools/measure.sh` in the experiments. Things only
  visible on the display (hardware sprites, copper effects) are checked
  on screenshots the tests take through the emulator's Lua
  (`Amiga.screenshot` in `game/tools/amiga.py`), or by the user looking.

## Decisions

### Display
- Low-res 320x256, 4 bitplanes, interleaved, double buffered, one fixed
  16-colour palette: black plus the 15 colours the arcade uses for
  enemies and text. Measured in MAME over stages 1-34 (`analysis/`).
- Blitter priority ("nasty") on. Game logic runs after the frame's
  drawing, not interleaved with it.

### Screen layout
- 224x256 playfield at the left, 16 px black gap, 80 px panel (10
  characters) at the right for score, high score and lives.
- The arcade's 224x288 screen loses its top two and bottom two text
  rows to the panel; all gameplay fits arcade lines 16-271.
- The buffer is 336x288: 16 hidden pixels at the left and 16 hidden
  rows above and below, so flyers need no clipping code. At the right
  edge the blit's first-word mask cuts off what would show in the gap.

### Rendering
- **Formation:** not 40 bobs. Each of its 5 rows is a pre-composed strip
  copied with one plain blit per frame; one strip is rebuilt per frame.
  No erase, no mask, and it repairs damage from flyer erases. Only the
  part of a strip with enemies in it is copied: from a blank word
  before the first to two blank words after the last, joined with what
  was copied at the two drawings before (each screen is drawn every
  other frame), so what an enemy leaves behind when it goes is wiped
  from both screens and an empty row then costs nothing.
  The formation is where the arcade's is, read from the arcade's own
  two tables (`HomeLoc`, `HomeX` in the state), which the flights read
  too. At rest the rows' tops are at 36 (bosses), 52, 64, 76 and 88, 12
  pixels apart below the bosses, and the columns 16 apart from x = 32.
  The arcade moves the columns up to 32 pixels either way and the bottom
  row up to 32 down (measured over the MAME trace); strips are sized for
  that. Butterflies and bees are 10 rows tall upright, so their strips
  are 14 lines (two blank lines each side, which overlap the next
  strip's when the rows are closed up); the boss strip is 18 with a
  blank line above and below. The blank lines wipe what a moving row
  leaves behind. A mask per row (`FormPresent`) says who is there.
- The formation moves as the arcade's does: it drifts sideways as a
  whole while the waves arrive (a pixel every 4 arcade frames, up to 32
  either way), then breathes (each column and row a pixel out or in
  every 4 frames by bit patterns from the ROM, 32 steps each way).
  `motion/formation.py` is the model: identical to the MAME trace's
  tables on all 33,016 frames of the 7 stages with a formation. The
  68000 version is `FormationTick`; the stage test checks it every
  frame. The drift is added when strips are drawn, so all rows move
  together; breathing reaches a row when its strip is recomposed, so a
  row can be up to 5 frames (2 pixels) behind. When breathing starts
  the pulse sound starts, and follows its direction. The user has
  confirmed the breathing looks right (2026-10-01).
- A strip only wipes its own old image if it moves a line or two. A
  cleared stage ends with the formation empty, so the jump back to the
  rest position leaves nothing behind. When a formation with enemies in
  it is taken away (a game is over: `StageIdle`), it is emptied where it
  stands, not put back at rest: its strips, rebuilt empty, are drawn
  once more where the enemies were and wipe them, at no extra cost. The
  next stage's start puts the formation at rest. (Wiping the whole
  playfield with the blitter was tried first: about 270 lines a screen.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mwulffn/AmiGalaga](https://github.com/mwulffn/AmiGalaga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

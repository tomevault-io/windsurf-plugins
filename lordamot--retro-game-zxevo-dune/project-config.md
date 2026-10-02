---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working in
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in
this repository.

## Project overview

This project ports the Mega Drive **Dune II - The Battle for Arrakis** to
the **ZX Evolution BaseConf** (Z80 at 14 MHz, 4 MB, a General Sound card
with 2 MB).  It is built as **`build/DUNE.DAT`** - the whole game in the
machine's paged-executable format (SPG), up to 4 MB, every byte the game
needs loaded into RAM pages at once - and shipped as **`build/dune.trd` +
`build/DUNE.DAT`**: the BaseConf firmware runs no SPG file, so a TR-DOS
disk it does boot carries a loader (`src/loader/loader.asm`) that reads
DUNE.DAT off the SD card itself.  There is no `dune.spg` any more; the
emulator's `spg` command loads DUNE.DAT (the same bytes).

The game runs in the ZX Evolution's **ATM EGA mode - 320x200 in sixteen
colours out of 64 - at 14 MHz**, double-buffered, which is what the whole
renderer is arranged around.  The Mega Drive's battlefield uses 31 colours
on a 320x224 screen; the port chooses sixteen and draws the rest as fixed
two-colour checkerboards (`tools/dune_art.py`).

Nothing is installed on the host.  The repo builds its own assembler, its
own **ZX Evolution emulator** (`tools/evo-emu/`, a scriptable frontend over
libxpeccy, since no buildable one exists here) and both reference machines'
libretro cores into `bin/`.

```
make build     # src/ -> build/DUNE.DAT + dune.trd (what a real machine runs)
make run       # play it (SDL2 window, sound; the ZX keys are the Mega Drive pad)
make run HOUSE=A MISSION=3   # ... straight into that battle (H A O, 1-9)
make verify    # start it headless and check from memory that a battle,
               # the front end, building and the sound card really work
make demo      # build straight into a battle and photograph it
make shot      # photograph the SPG five seconds in
make sega      # the Mega Drive's music and effects as General Sound .mod files

make toolchain # rebuild the assembler, the emulator and both cores
```

Key documentation: `.claude/docs/port.md` (the port's shape: the SPG, the
pages, the windows, the record layouts), `.claude/docs/port-code.md` (how
the Z80 code is organised: banks, `FCALL`, the library, the script
machine, who owns what), `.claude/docs/progress.md` (state of the port),
`.claude/docs/platform.md` (the machine: video, the memory manager, the
shadow ports, the palette, starting an SPG), `.claude/docs/graphics.md`
(the planes, the renderer's dirty cells, sprites, the front end's
pictures), `.claude/docs/gamedesign.md` (the game as the port plays it, and
where it deliberately differs from the cartridge), `.claude/docs/build.md`
(the pipeline and the checks), `.claude/docs/emulator.md` (the emulator,
its script language and its profiler), `.claude/docs/sound.md` (the
Mega Drive's sound on the General Sound card), `.claude/docs/tools.md`,
`.claude/docs/sega-internals.md` and `.claude/docs/originals.md` (the
deconstructions), `.claude/docs/sega-handover.md` (how the deconstruction
became the specs the port is written from), `.claude/docs/nes-tools.md`
(the two reference machines and their script language).  Follow
`.claude/rules/guideline.md` and `.claude/rules/git.md`.

## Traps worth remembering

- **The Kempston port answers only with the shadow ports shut.**  The
  manual marks `#xx1F` "noshad": with `$BF` bit 0 set, which the game
  keeps for the manager and the palette, port `$1F` is the floppy
  controller's status (TR-DOS mode on) or nothing at all - and nothing at
  all reads as the byte the video is fetching, so the pad saw random
  presses every frame on a real machine: the intro skipped before its
  tune, menus wandering, a restart chosen.  `pad_read` shuts the shadow
  ports for the one read (`src/stub/input.asm`).  The emulator answers
  the floppy status there and never showed it.
- **The BaseConf firmware does not run SPG files.**  Its file browser
  knows TRD, SCL, FDI and TAP; an .spg is "unknown", and choosing one
  stages it into RAM from page #F4 downwards until it wraps into the
  firmware's own pages - a black screen on a real machine.  SPG is a
  TS-Conf format (TS-BIOS, Wild Commander).  So the game is delivered as
  `dune.trd` + `DUNE.DAT`: the disk's loader reads the SPG off the card
  over the SPI port (`src/loader/loader.asm`, `tools/dune_trd.py`), and
  `make verify` boots it that way, from FAT16 and FAT32 card images
  (`tools/sd_image.py`, `bin/evo/evo-run --sd`).
- **No block may live in pages #E0-#FF.**  The SPG format allows #00-#DF
  because every loader keeps itself above that (the firmware's resident
  code is in FE, Wild Commander's runner runs from FE).  The Tutorial once
  sat in 249-254 and would have overwritten any loader; `tools/spg.py`
  now refuses such a page and the Tutorial is in 221-223, 0, 4, 6.
- **libxpeccy's SD card never left multi-block mode.**  Its CMD12 branch
  does not clear the flag and CMD17 falls into CMD18's case, so after the
  firmware's CMD18/CMD12 reads a program's single-block read never ends
  and its next command is swallowed - the loader "read" zeros and died on
  the MBR.  `tools/build_toolchain.py` patches `sdcard.c`; a real card is
  fine.
- **Bit A9 of port `xx77` holds TR-DOS mode on.**  `PORT_77` was `$8177`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lordamot/retro-game-zxevo-dune](https://github.com/lordamot/retro-game-zxevo-dune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->

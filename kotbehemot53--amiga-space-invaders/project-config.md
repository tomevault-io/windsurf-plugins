---
trigger: always_on
description: Space Invaders clone for a stock Amiga 500 — OCS, PAL, 512 KB chip + 512 KB
---

# CLAUDE.md

Space Invaders clone for a stock Amiga 500 — OCS, PAL, 512 KB chip + 512 KB
slow RAM, Kickstart 1.3+. Entire game is one file: `main.asm` (~1900 lines
of 68000 assembly, vasm mot syntax). No OS calls after startup; full
hardware takeover, restores system state on exit (left mouse button quits).

## Build & run

```powershell
.\scripts\build.ps1   # vasm + vlink -> uae/dh0/invaders
.\scripts\run.ps1     # FS-UAE as A500 (512/512, kick13), boots game via startup-sequence
```

Bash equivalents for Linux/macOS/Git Bash: `./scripts/build.sh`,
`./scripts/run.sh` (auto-detect the `.exe` suffix).
`./scripts/get-tools.sh` downloads the toolchain bundle for the current
OS into `tools/`. All scripts resolve the project root from their own
location, so they work from any cwd.

Manual equivalent:

```
tools\vasmm68k_mot.exe -m68000 -Fhunk -linedebug -o build\main.o main.asm
tools\vlink.exe -bamigahunk -Bstatic -o uae\dh0\invaders build\main.o
```

- Toolchain lives in `tools/` (prb28/vscode-amiga-assembly-binaries,
  `windows_x64` branch: vasm 1.9, vlink 0.17, FS-UAE, ADF tools) but is
  **gitignored**, as are `roms/`, `build/` and the built exe — a fresh
  clone needs `scripts/get-tools.sh` (or the README's PowerShell steps)
  plus a Kickstart image before it builds/runs.
- **FS-UAE must be launched with working directory = `tools/`** — it loads
  `fs-uae.dat` relative to cwd and silently dies early otherwise.
- `roms/kick13.rom`: Kickstart 1.3 image, not redistributable (copied from
  `E:\_Moje_dydy\Documents\Projekty\amiga_exp\amiga-mrzx\roms`).
- VSCode: extension `prb28.amiga-assembly`; `.vscode/settings.json` points
  `amiga-assembly.binDir` at `tools/`, F5 = FS-UAE + GDB remote debugger,
  Ctrl+Shift+B = build. "create ADF" task -> `build/invaders.adf`.
- Emulator keyboard-as-joystick: cursor keys + Right Ctrl/Alt = fire
  (`--joystick_port_1=keyboard`).

## Repo layout

```
main.asm            the whole game
scripts/            build/run (.ps1 + .sh) and get-tools.sh
doc/                walkthrough + line-by-line deep dives
.vscode/            Amiga Assembly extension config (tracked)
uae/dh0/            emulator hard drive; s/startup-sequence tracked, exe not
tools/, roms/       gitignored: toolchain + Kickstart (see Build & run)
build/              gitignored: intermediate objects, ADF output
```

## Documentation

`doc/CODE_WALKTHROUGH.md` = full architecture/hardware explainer for
high-level-language folks; `doc/dive-*.md` = line-by-line readings of
the key routines (mainloop, copper, blitter, aliens, bullet/BCD,
audio). Keep them in sync when changing the routines they quote.

## Source layout (single file, section order matters)

`main.asm` top-to-bottom:

1. Register equates, layout constants, `WAITBLT` macro
2. `section code` — startup/restore, main loop (`MainLoop`), copper list
   builder (`BuildCopper`), states (title/play/death/gameover/wave),
   formation logic, blitter helpers, CPU drawing, text, audio routines
3. CPU-read tables kept **in the code section** so `(pc)`-relative `lea`
   works: texts, font, palette (`PalTab`), gradient (`GradTab`), colour
   bands (`BandTab`), `AlienGfxTab`, `RowPts`/`UfoPts` (BCD), `SprPtrTab`
4. `section chipdata,data_c` — sprites (`PlayerSpr`, `UfoSpr`, `BlankSpr`),
   blitter gfx (aliens, `ExplGfx`, `LifeIcon`, `CellMask`)
5. `section data,data` — `HiTab` (top-5) + `HiScore` (persist across games)
6. `section bss,bss` — all game variables (loader zeroes them)
7. `section chipbss,bss_c` — 3 bitplanes, `CopBuf` (4 KB), audio buffers

## Display architecture

- 320x256 lowres, 3 bitplanes, non-interleaved. BPLCON0=$3200, BPLCON2=$0024
  (sprites in front). DIWSTRT $2c81 / DIWSTOP $2cc1, DDF $38/$d0.
- **Plane 0** = all game objects (aliens, bullets, bombs, shields, ground,
  life icons). Colour comes from copper: `BandTab` retints COLOR01 per
  screen region (rainbow alien rows, green shield/player zone) — arcade
  "film gel" trick.
- **Plane 1** = text/HUD only. **Plane 2** = starfield (COLOR04 twinkled by
  CPU poking a word inside `CopBuf` via `TwinkPtr`).
- Copper list built at runtime by `BuildCopper` into `CopBuf`: bpl+sprite
  pointers, palette, then per-4-lines COLOR00 gradient with a `$ffdf` wait
  crossing raster line 255. Sprite pointers must be rewritten every frame
  (copper does it); sprite movement = rewriting pos/ctl words in sprite data.
- Main loop is vblank-locked via `WaitVBL` (two-phase wait on VPOSR line 303,
  includes bit 8 through the `$1ff00` mask — don't simplify it away).

## Game logic conventions

- **a5 = $dff000 always** after takeover. Every routine assumes it.
- Screen coords: x 0-319, y 0-255; sprite hardware pos = x+$81, y+44
  (`SetSprPos` handles 9-bit V for player at y=216 -> raster 260).
- Formation: 5 rows x 11 cols, 16px cells, `AlienTab` = 55 alive-bytes.
  Whole band erase + redraw only on move tick (`MoveDelay` shrinks with
  `AlienCnt` and Level). Row types via `RowType`, 2 anim frames.
- Blitter objects are 16px gfx stored as 2 words/row (image + zero pad) so
  A-shift never needs masking. `BlitObj16` = OR-blit (LF $FA), `BlitCell` =
  cookie-cut replace with `CellMask` (LF $F2) so neighbours survive.
- Bullets/bombs are CPU-drawn 2px columns (`DrawVLine`/`EraseVLine`), x

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kotbehemot53/amiga-space-invaders](https://github.com/kotbehemot53/amiga-space-invaders) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

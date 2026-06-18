---
trigger: always_on
description: Architecture / invariants / gotchas for the **emulator side** of POM1. User walkthrough → `README.md`; open work → `TODO.md`; history → `git log`.
---

# CLAUDE.md

Architecture / invariants / gotchas for the **emulator side** of POM1. User walkthrough → `README.md`; open work → `TODO.md`; history → `git log`.

**Contents:** [Overview](#project-overview) · [Build](#build--run) · [CLI](doc/CLI.md) · [WASM / cc65](#wasm-build) · [Architecture](#architecture) · [MMIO / CPU](#key-implementation-details) · [Memory map](#memory-map) · [Platforms](#platform-notes) · [Testing](#testing) · [Version bump](#version-string-locations)

- Apple 1 software (BASIC, SID tunes, microSD shell tools, games) → **`dev/APPLE1DEV.md`** + **`dev/Programming_Apple1_ASM.md`**.
- **CLI flags** (full table) → **`doc/CLI.md`** · implementation **`CliDispatcher.cpp`**.
- 6502 ASM sources for every shipped program → **`dev/`** (`lib/{apple1,m6502,tms9918,hgr,sokoban}/`, `projects/<name>/`, `cc65/`). Compiled `.bin`/`.txt` land under `software/<dir>/` — that is what POM1 loads. Release bundles omit `dev/`; clone the repo for sources.

## Project Overview

Apple 1 emulator (Dear ImGui, MOS 6502 + display + keyboard + ACI cassette) plus expansion cards: Uncle Bernie's GEN2 HGR, P-LAB A1-SID (6581/8580), TMS9918, microSD (65C22+ATMEGA), **IEC daughterboard** (1541 drive over SN7406 on microSD's spare VIA pins), MODEM BBS (65C51+TCP), Terminal Card, A1-IO & RTC (65C22+ATMEGA32+DS3231), Juke-Box, CodeTank, Rich Dreher's CFFA1, SWTPC GT-6144 (1976) and PR-40 (1976, Jobs' *Interface Age* mod). Linux / macOS / Windows / Web (Emscripten).

**Preset truth:** `--preset N` indexes `kMachinePresets[]` in `MainWindow_Presets.cpp` (RAM size, cards, BASIC type) — 14 presets (0–13). The README preset table must stay in lockstep with that array. **Invariants:** A1-SID and A1-AUDIO SE share ONE preset (#6) — the card variant + I/O window (`$C800-$CFFF` vs `$CC00-$CC1F`) is picked at runtime from **Settings → "A1-SID version & addresses"** (also lists the 29 SID register addresses); **no TMS9918 preset without CodeTank** (daughterboard rule, see below).

## Build & Run

```bash
./setup_pom1.sh             # one-time deps (Linux/macOS)
cd build && cmake .. && make # build → build/POM1
./run_emulator.sh            # runs from repo root
```

Windows: `setup_pom1.bat` + vcpkg + `cmake --build . --config Release`. `compile_commands.json` symlinked for clangd.

### CLI

Full flag table (phases A/B/C, every verb, telnet helpers) → **[`doc/CLI.md`](doc/CLI.md)**. Implemented in **`CliDispatcher.cpp`**. Malformed verbs log `[CLI] ERROR:` and exit 1.

### WASM build

```bash
source /path/to/emsdk/emsdk_env.sh
mkdir -p build-wasm && cd build-wasm
emcmake cmake .. && emmake make -j$(nproc 2>/dev/null || sysctl -n hw.ncpu)
emrun POM1.html
```

MEMFS preload (`if(POM1_IS_WASM)` in `CMakeLists.txt`) mounts `roms/ pic/ fonts/ software/ sdcard/ cassettes/` and the single `cfcard/cfcard.po` (other `.po` extras are desktop-only, >140 MB). Rebuild WASM after any change under those trees or `build-wasm/shell.html`. POST_BUILD copies `pic/icon.png` to `build-wasm/pic/` as an HTTP-visible favicon (MEMFS unreachable to the browser). Desktop `Memory` ctor probes `sdcard`, `../sdcard`, `../../sdcard` and `cfcard/cfcard.po`.

### Assembling programs (cc65)

Sources in `dev/`. Per-project Makefiles call `ca65` + `ld65`, then `python3 emit_*_txt.py` lands `.bin` + Woz-hex `.txt` under `software/<original_dir>/`. `emit_woz.py extra_zones` bundles raw blobs (e.g. an 8 KB HGR image at `$2000`) as extra hex zones in the `.txt`. Makefiles pass `-I ../../lib/<lib>` so `.include "apple1.inc"` works. GEN2 dev guide ("Bernie SDK") → **`doc/GEN2_RELEASE.md`**; reference demo → `dev/projects/a1_crazycycle/`.

**Linker configs** (`dev/cc65/`): `apple1_4k.cfg` (text / TMS9918), `apple1_gen2.cfg` (HGR — reserves `$2000-$3FFF`), `codetank.cfg` (standalone CodeTank ROM `$4000-$7FFF`), `pom1_fantasy.cfg` (Multiplexing Fantasy). Sokoban (`dev/projects/games_sokoban/apple1_sok_{4k,8k,hgr}.cfg`) defines `LEVELBUF` (zp `$0020`) + `STATEGRID` (bss); use `.segment "LEVELBUF": zeropage` to force `zp,X`.

**Reusable libraries** (`dev/lib/`): `apple1` equates, `m6502` math, `tms9918` driver, `hgr` tables, `games/{chess,sokoban}` (shared engine + level data), plus:

- **`gen2`** — GEN2 release equates + HST0 beam-sync (`gen2_waitvbl` / `gen2_beam_lock`); see `doc/GEN2_RELEASE.md`.
- **`gen2c`** — C runtime. `gen2_hgr_clear` (fast page/Y-indexed loop — the naïve 16-bit pointer loop was ~20× slower + blanked the screen for seconds), `gen2_hgr_row`, `gen2_hgr_puts` (ASCII via the embedded Beautiful Boot 8×8 font, pixel-doubled H+V → solid white HIRES, no NTSC artifacts). **Hot paths are hand-written 6502 in `gen2_blit.s`** — the pure-C version did per-pixel `x/7`/`x%7` software division (no 6502 DIV), costing millions of cycles per text line:
  - `gen2_blit_glyph` — text inner loop, ~10× faster (caller passes start col/mask; asm advances them incrementally, zero per-pixel division).
  - `gen2_fill_rect_asm` behind `gen2_hgr_fill_rect` — fast byte-aligned rectangle erase, beats per-pixel `gen2_hgr_unplot`.
  - `gen2_plot_asm`/`gen2_unplot_asm` behind `gen2_hgr_plot`/`unplot` — ~4× faster per pixel (byte col/mask from the `gen2_col7`/`gen2_mask7` LUTs, not division).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habib256/POM1](https://github.com/habib256/POM1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A research and port project for Konami's *Snatcher* (Sega CD, 1994). `engine/` is a Mega CD core that runs the game's own 68000 code, with the Sega CD BIOS emulated in host code (no BIOS file needed). It has an SDL3 front-end for Mac and Windows. Strategy and decisions live in `docs/` (`STRATEGY.md`, `ADR-001`, `ADR-002`); `docs/JOURNAL.md` is the append-only log of confirmed reverse-engineering facts. Add new findings there.

## Hard rules

- **Never commit disc images, BIOS dumps, extracted data, decompiled/disassembled listings, MAME captures, or generated translated C++.** `sega-cd-eng/`, `pcengine-cd-jpn/`, `extracted/`, `work/`, `dist/` and `engine/tests/lift_cases.inc` are gitignored. Binaries in `dist/` embed disc-derived code and must not be shared.
- The user supplies their own disc in `sega-cd-eng/` (`Snatcher (Sega CD) (U)-redump.cue/.bin`). Tools read it; nothing from it enters the repo.
- The safety hook blocks `rm -rf`. Use a fresh build directory or `mv` instead of deleting.
- Always build with Apple clang on macOS (`CC=/usr/bin/clang CXX=/usr/bin/clang++`). A `~/.nix-profile` gcc fails to link against the macOS SDK. A cached CMake dir configured with the wrong compiler must be replaced, not reconfigured.

## Environment

`nix develop` (flake.nix) provides toolchain, SDL3, Ghidra, mingw-w64, ffmpeg. `mise run <task>` wraps the workflow (`mise tasks` lists them): `setup`, `extract`, `build`, `test-lift`, `package:mac`, `package:windows`, `run`. The Python env is `.venv` (capstone); the repo's tools use it (`.venv/bin/python`).

## Build, run, test

```sh
# Extract code blobs from your disc (-> extracted/code/*.bin), needed for the translated build
.venv/bin/python tools/extract_code.py "sega-cd-eng/Snatcher (Sega CD) (U)-redump.bin"

# Plain interpreter build (no disc-derived code), CMake from engine/
CC=/usr/bin/clang CXX=/usr/bin/clang++ cmake -S engine -B engine/build-interp && cmake --build engine/build-interp -j

# Translated build: generates C++ from extracted/code at build time (needs ovl_909.bin, see below)
cmake -S engine -B engine/build -DSNATCHER_TRANSLATE_DIR=$PWD/extracted/code && cmake --build engine/build -j

# Run (SDL window; --justifier makes the mouse the light gun on port 2)
engine/build/snatcher "<disc>.cue" [--justifier]

# Packages (rebuild after engine changes; both regenerate the translation from extracted/code)
tools/package/mac_app.sh              # dist/Snatcher.app
tools/package/windows_cross.sh        # dist/windows/snatcher.exe (mingw-w64, static SDL3)

# Differential test of the translator: every instruction encoding in the disc code vs Musashi
python3 tools/lift/gen_tests.py engine/tests/lift_cases.inc 20000
cmake --build engine/build --target lift_diff && engine/build/lift_diff 8     # must report 0 failing cases
```

The `$28000` overlay is compressed on disc, so its translation input is a runtime dump:
`SCD_NO_TRANSLATE=1 SCD_DUMP_PRG=1500:28000:20000:extracted/code/ovl_909.bin engine/build-interp/scd_headless "<disc>.cue" --frames 1500 --out work/out`.

`scd_headless` (no window) is the main debugging tool: `scd_headless <cue> --frames N --out DIR [--ppm-every N] [--press FRAME:BUTTONS:HOLD] [--wav f]`. Buttons are `U D L R B C A S`. Frame numbers count from the start of the run, and the game's first frame has `$FFE020 >= 1`. Useful env vars: `SCD_STOP_E020=<hex>` (stop at a game frame counter), `SCD_STATE=1`, `SCD_DEBUG=1`, `SCD_TRACE=cpu:frame:count`, `SCD_TRACE_BIOS=1`, `SCD_WATCH=<main RAM addr>`, `SCD_VDPLOG`, `SCD_VDPPORT`, `SCD_DUMP_RAM=1`, `SCD_VRAM=1`, `SCD_DUMP_AT=f1,f2`, `SCD_NO_TRANSLATE=1`, `SCD_LIFT_VERIFY=1`, `SCD_MUTE=ym,pcm,psg,cdda`, `SCD_GUN=x,y`, `SCD_BRAM_SELFTEST=1`.

`tools/test_intro_skip.sh [scd_headless]` is the end-to-end regression for the intro skip / Act 1 hang (boots, skips the intro, expects the Junker HQ scene; works with `SCD_NO_TRANSLATE=1` too). Debug aids: `--log` / `SNATCHER_LOG=1` (app) or `SCD_LOG=file` (headless) write a persistent log, `F10` snapshots state, `SCD_COMMLOG=1` logs Gate Array comm writes, `SCD_PRGPOLL=addr,..` watches Sub PRG RAM bytes. In headless runs build `--press` lists in bash (zsh does not word-split an unquoted `$VAR`, which silently drops every press).

There is no linter. Correctness is judged against MAME (below).

## Architecture

**The game is two 68000 programs, not a script VM.** The Main CPU IP (`$FF0000`, from disc `0x200..0x6800`) is a hardware driver: VDP, controllers, a YM2612 driver, and a V-INT handler that runs most of the game via state machines. The Sub CPU runs the engine: SP at `$6000` (boot loader, ISO9660 reader), `SUBCODE.BIN` at `$D400` (script/game logic, CD-DA, PCM streaming), and overlays loaded to `$28000`. They communicate through Gate Array comm registers (`$A12010+`/`$A12020+`, flags `$A1200E/F`) and Word RAM mailboxes (`$B6xxx` on the Sub side). See `docs/DISC_LAYOUT.md` and the journal for addresses.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bgyss/snatcher-port](https://github.com/bgyss/snatcher-port) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

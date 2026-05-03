---
trigger: always_on
description: This repository contains a complete SNES ROM project that recreates Dragon's Lair arcade as a full-motion-video game using the MSU-1 enhancement chip.
---

# GitHub Copilot Instructions for SNES Super Dragon's Lair Arcade

This repository contains a complete SNES ROM project that recreates Dragon's Lair arcade as a full-motion-video game using the MSU-1 enhancement chip.

## Project Overview

**Super Dragon's Lair Arcade** is an FMV retheme of RoadBlaster for SNES, targeting real NTSC hardware with MSU-1 audio/video support on SD2SNES/FXPAK Pro. The project includes:
- 65816 assembly code for SNES ROM
- SPC700 assembly for audio processor
- Python-based asset processing pipeline
- MSU-1 video/audio packaging tools
- Build system using Make and WLA-DX 9.3 assembler

## Critical Context

### Build System
- **Platform:** Linux/WSL (Ubuntu) required
- **Assembler:** WLA-DX 9.3 (pre-built binaries in `tools/wla-dx-9.5-svn/` — despite the directory name, the actual version is 9.3; v9.4+ breaks the build)
- **Primary Build:** `wsl -e bash -c "cd <wsl-project-root> && make clean && make"`
- **Fast Rebuild:** `wsl -e bash -c "cd <wsl-project-root> && make"` (skip clean if only assembly files changed)
- **Output:** `build/SuperDragonsLairArcade.sfc` (1 MB SNES ROM, 16 banks, HiROM+FastROM)
- **Warning:** `make clean` deletes `data/chapters/` — wipes all extracted video frames

### Python Toolchain (Python 3.10+)
All Python scripts are **Python 3 compatible**. Key tools:
- `tools/generate_msu_data.py` - **MSU-1 video pipeline orchestrator** (ffmpeg → superfamiconv → tile reduction → .msu)
- `tools/animationWriter_sfc.py` - Default graphics animation converter (uses superfamiconv)
- `tools/animationWriter.py` - Legacy animation converter (uses gracon.py)
- `tools/gracon.py` - Legacy Python graphics converter (slower, functional)
- `tools/gfx_converter.py` - Unified wrapper for superfamiconv or gracon
- `tools/img_processor.py` - Image resizing/quantization to SNES specs
- `tools/mod2snes.py` - MOD music to SPC700 format converter
- `tools/msu1blockwriter.py` - Packages MSU-1 data files
- `tools/msu1pcmwriter.py` - Converts WAV to MSU-1 PCM format
- `tools/xmlsceneparser.py` - DirkSimple XML chapter/event processor

Install dependencies: `pip install -r requirements.txt` (Pillow, NumPy)

### Asset Pipeline

#### Graphics (SNES native format)
1. **Backgrounds:** 256x224 pixels, 16 colors (4bpp), processed via `img_processor.py`
2. **Sprites:** 4bpp with 2 palettes, optimized for OAM
3. **Direct Color:** 8bpp mode for HUD overlays
4. **Conversion:** Build system auto-converts `.png` files using `animationWriter_sfc.py`
5. **Performance:** superfamiconv is ~100x faster than gracon.py

#### Audio
- **SPC700:** 6 active sound effects in BRR format (~42 KB of 57.5 KB sample budget)
- **MSU-1:** CD-quality PCM audio for video tracks
- **Registration:** All sounds must be registered in `src/object/audio/spcinterface.h`

#### Video (MSU-1)
- 516 chapters defined in `data/events/*.xml` (DirkSimple game data format)
- Pipeline: `generate_msu_data.py` orchestrates ffmpeg → superfamiconv → tile reduction → msu1blockwriter.py
- Output: ~568 MB `.msu` file with 256x192 frames, 16 colors, 512 tiles per frame
- Video timing: 23.976 fps (source: Daphne `.m2v` segments in `data/laserdisc/segments/`)

### File Structure
```
src/           - 65816 assembly source code
data/
  backgrounds/ - PNG images for screens (9+ backgrounds)
  sprites/     - PNG sprites for arrows/effects (16 sprite types)
  sounds/      - WAV/BRR audio files (11 WAVs, 6 active in SPC build)
  chapters/    - Generated chapter scripts and data (from xmlsceneparser.py)
  events/      - XML chapter scripts (516 files, DirkSimple game data)
tools/         - Python asset converters and build utilities
build/         - Generated output (ROM, .msu, intermediate files)
schwag/        - Marketing materials and documentation
```

### Code Style and Conventions

#### Assembly (65816/SPC700)
- Follow WLA-DX syntax
- Use `.65816` extension for SNES CPU code
- Use `.spc700` extension for audio processor code
- Include files use `.inc` extension
- Labels starting with `_` are LOCAL to compilation unit — cannot cross `.o` file boundaries
- `.def` cannot redefine — second `.def X Z` after `.def X Y` is silently ignored; use `.redefine`
- Anonymous labels `+`, `++`, `+++` are distinct tiers — `bra ++` targets next `++`, not second `+`
- Use named labels over anonymous when macro expansions (NEW, CALL) appear between branch and target
- For long forward branches: `bne _skip / jmp far_target / _skip:` pattern

#### Python
- **Python 3.10+ only** (all tools migrated from Python 2)
- Follow PEP 8 style guidelines
- Handle binary/text properly (bytes vs strings)
- No silent try/except wrappers — fix root causes

### Current Project Status

#### Completed
- Full MSU-1 video pipeline (516 chapters, ~568 MB .msu file)
- All 29 scenes playable across 9 levels
- Title screen with menu system (Start Game, Options, Sound Test, Scene Select)
- Attract mode and losers screen
- Chapter/event system with data table architecture (36+ event classes)
- SPC700 audio system (6 sound effects)
- Dragon's Lair themed backgrounds, sprites, and UI
- WLA-DX 9.3 build system working
- Python 3 migration complete for all tools

#### Remaining Work

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [astrobleem/SNES-SuperDragonsLairArcade](https://github.com/astrobleem/SNES-SuperDragonsLairArcade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

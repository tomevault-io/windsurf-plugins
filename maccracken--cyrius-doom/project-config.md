---
trigger: always_on
description: > **Core rule**: this file is **preferences, process, and procedures** — durable rules that change rarely. Volatile state (current version, binary sizes, dep pins, in-flight slots, gates, consumers, recent releases) lives in [`docs/development/state.md`](docs/development/state.md), refreshed every release. Historical release narrative lives in [`docs/development/completed-phases.md`](docs/development/completed-phases.md). Do not inline state here — inlined state rots within a minor.
---

# Cyrius DOOM — Claude Code Instructions

> **Core rule**: this file is **preferences, process, and procedures** — durable rules that change rarely. Volatile state (current version, binary sizes, dep pins, in-flight slots, gates, consumers, recent releases) lives in [`docs/development/state.md`](docs/development/state.md), refreshed every release. Historical release narrative lives in [`docs/development/completed-phases.md`](docs/development/completed-phases.md). Do not inline state here — inlined state rots within a minor.

---

## Project Identity

**cyrius-doom** (homage to id Software's DOOM, 1993) — Clean-room DOOM engine in Cyrius. Direct framebuffer, no libc, no SDL, kernel syscalls only.

- **Type**: Standalone game binary / kernel demo
- **License**: GPL-3.0-only (clean-room implementation from documented specs)
- **Language**: Cyrius (toolchain pinned in `cyrius.cyml [package].cyrius` — `cycc 6.1.29` at the time of writing; canonical pin is the file)
- **Version**: `VERSION` at project root is the source of truth (referenced via `version = "${file:VERSION}"` in `cyrius.cyml`). Do not inline the number here.
- **Genesis repo**: [agnosticos](https://github.com/MacCracken/agnosticos)
- **Standards**: [First-Party Standards](https://github.com/MacCracken/agnosticos/blob/main/docs/development/planning/first-party-standards.md) · [First-Party Documentation](https://github.com/MacCracken/agnosticos/blob/main/docs/development/planning/first-party-documentation.md)
- **Philosophy**: [AGNOS Philosophy](https://github.com/MacCracken/agnosticos/blob/main/docs/philosophy.md)

## Goal

Own a clean-room DOOM engine in Cyrius — runnable on /dev/fb0, on AGNOS, and eventually on bare metal. No id Software code copied; everything from documented specs. Proves Cyrius can drive a real-time renderer at original-DOOM fidelity without libc, FPU, or external runtime.

## Current State

> Volatile state lives in [`docs/development/state.md`](docs/development/state.md) — current version, binary sizes, dep pins (bsp / vani tags), in-flight slot status, last-green gates, known-issue workarounds. Refreshed every release.
>
> Historical release narrative lives in [`docs/development/completed-phases.md`](docs/development/completed-phases.md). Per-release detail lives in [`CHANGELOG.md`](CHANGELOG.md).
>
> Forward-facing slots live in [`docs/development/roadmap.md`](docs/development/roadmap.md).

## Scaffolding

Project was scaffolded manually; subsequent modernization passes match the patra/vani/sakshi/mihi convention (single `cyrius.cyml`, `${file:VERSION}` template, patra-style CI installer). Do not hand-roll new structure if `cyrius init` covers it — fix the tool, then re-propagate.

## Quick Start

```sh
# Build (requires the toolchain pinned in cyrius.cyml)
cyrius build src/main.cyr build/doom

# Release build: NOPs dead functions in-place (used by release.yml)
CYRIUS_DCE=1 cyrius build src/main.cyr build/doom

# Run (requires DOOM1.WAD — see scripts/get-wad.sh)
./build/doom wad/DOOM1.WAD              # interactive (/dev/fb0 or GTK bridge)
./build/doom wad/DOOM1.WAD --ppm        # game screenshot mode (headless)
./build/doom wad/DOOM1.WAD --ppm-menu   # menu screenshots
./build/doom wad/DOOM1.WAD E1M3 --ppm   # specific map

# Download shareware WAD (one-time, not in repo)
sh scripts/get-wad.sh wad

# Test
cyrius test tests/doom.tcyr                                  # WAD-free subset
cyrius build tests/doom.tcyr build/test_doom && \
  ./build/test_doom wad/DOOM1.WAD                            # full suite

# Fuzz
cyrius build fuzz/fuzz_fixed.cyr build/fuzz_fixed && ./build/fuzz_fixed
cyrius build fuzz/fuzz_wad.cyr   build/fuzz_wad   && ./build/fuzz_wad

# Bench (also appends a row to bench-history.csv)
sh scripts/bench-history.sh wad/DOOM1.WAD

# One-shot
sh scripts/run.sh
```

## Architecture (durable — module map)

```
src/
  main.cyr        — entry, game loop (35Hz tick), --ppm screenshot mode
  fixed.cyr       — 16.16 fixed-point math, native `>>>` arithmetic right shift
  tables.cyr      — 1024-entry sine table (Bhaskara I), atan2, trig wrappers
  wad.cyr         — WAD parser (IWAD/PWAD, directory, lump read/cache)
  framebuf.cyr    — 320x200 palette-indexed framebuffer, PPM output
  map.cyr         — vertices, linedefs, sidedefs, sectors, segs, subsectors, BSP nodes, things
  texture.cyr     — wall texture compositing from patches, flat cache, patch LRU cache
  render.cyr      — BSP traversal, textured wall columns, COLORMAP lighting, visplane spans, sky
  sprite.cyr      — thing sprites: distance sort, scale, clip to walls, sector lighting
  input.cyr       — terminal raw mode, WASD + arrows, bitmask action flags
  player.cyr      — movement, wall sliding collision, step height, ceiling check
  tick.cyr        — 35Hz timer via clock_gettime + nanosleep

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MacCracken/cyrius-doom](https://github.com/MacCracken/cyrius-doom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->

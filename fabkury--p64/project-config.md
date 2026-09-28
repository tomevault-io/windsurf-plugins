---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

p64 is a desktop 64x64 RGB LED matrix: Waveshare's ESP32-S3-RGB-Matrix driver board
plugged onto their RGB-Matrix-P2-64x64 panel, in a 3D-printed tabletop shell. Three
folders of work live here, each with its own README that is the detailed reference:

- `firmware/`: the p64 product firmware, started from zero on 2026-09-19. Its behaviour is
  fixed by the product specification `docs/spec/p64-spec.md` (settled with the user on
  2026-09-19, settings and limits tables included) and the decisions in `docs/adr/`; the
  vocabulary is `CONTEXT.md`. Its design is `firmware/docs/architecture.md`, its state of
  progress `firmware/docs/PROGRESS.md` (read that first when resuming), its API
  `firmware/docs/api.md`. The "Firmware" section below has the commands. Do not carry
  code or patterns over from the hardware tests on your own initiative.
- `firmware/reference/hardware-tests/`: the former `firmware/` (renamed on 2026-09-19,
  moved under `firmware/reference/` on 2026-09-24; git history follows both moves,
  `git log --follow` works on its files): ESP-IDF v5.5 test firmware, C++20,
  directly on the `esphome/esp-hub75` DMA driver (vendored and patched under
  `firmware/reference/hardware-tests/components/esp-hub75`). No Arduino, no LVGL, no Waveshare BSP (only
  its pin map was reused). It is the technical reference for the product firmware (what
  the hardware taught: pin map, driver patch, frame pacing, GDMA, network, microSD), not
  its architectural reference. The "Hardware tests" sections below describe it.
- `enclosure/`: the OpenSCAD shell, versioned as separate `.scad` files with outputs
  under `enclosure/output/vN/`.
- `docs/video/`: the p64b concept video pipeline (2026-09-24): `pieces.scad` exports the
  enclosure source's mock-ups, `bake_leds.py` the panel frames, `scene.py` builds and
  renders in Blender 5.2 (Cycles, GPU), `compose.py` adds bloom, captions and the end
  card, `storyboard.py` holds every time; `make.ps1` runs it all. `build/` and the MP4s
  are git-ignored; its README has the steps.
- `docs/hardware/`: wiring projects for the two rotary encoders of p64b
  (`encoders-solderless.md`, `encoders-soldered.md`, schematics drawn by
  `tools/draw_encoders.py` with schemdraw),
  written 2026-09-21 before any encoder was wired; the bench records in them are blank
  until measured.

The repository README is written for newcomers (what p64 is, photos under `docs/images/photos/`,
the parts list with dated Waveshare prices, an honest status) and `docs/build-your-own.md`
is the step-by-step build guide; keep both truthful when the status changes (first
release, v7 printed, encoders wired).

p64 has two variants, supported for good (decided 2026-09-22): **p64a**, solder-less,
without rotary encoders; **p64b**, with soldering, with the two rotary encoders. The
enclosure has one source and two variant files (`enclosure/src/p64_enclosure_v7a.scad`
and `_v7b.scad`), outputs under `enclosure/output/p64a/` and `p64b/`; the firmware is
shared. Anything new that touches the encoders is a p64b matter.

Also: `firmware/reference/` is git-ignored upstream clones (the one tracked exception is
`hardware-tests/`, above): Waveshare's example repo,
`p3a/` (the user's production ESP32-P4 pixel-art player, github.com/fabkury/p3a, the
reference for module boundaries, web UI and Makapix client) and `makapix/` (the Makapix
Club server, github.com/fabkury/makapix, the device contract in its `docs/player/` and
`docs/mqtt-api/`); re-clone commands are in `firmware/README.md`. `firmware/private/` is
the **private area** (ADR 0012, since 2026-09-24): a separate, unpublished repository,
github.com/fabkury/p64-private, mounted there and git-ignored here (a local pre-commit
hook refuses its paths); the user's own components, first a private source of channels.
The public build discovers it when present (`firmware/CMakeLists.txt`: components,
`sdkconfig.private`, version `+private`; `main.cpp` calls `p64::priv::start()` under
`CONFIG_P64_PRIVATE`; `tests/host/run.py` reads its manifest) and is the public firmware
without it, which is what CI builds; `tools\idf.ps1 -B build_public -DP64_NO_PRIVATE=ON
build` checks that parity locally. Nothing public may depend on it (no public symbol
whose only user is private), and its RAM costs are measured by hand. Channel sources
other than the card reach the show only through `content::Provider`
(`p64/content/provider.hpp`, the registry, kind `external` = `<provider>:<channel>`);
Makapix Club is the first provider and the private source is written as the second.
`prompt/pNNN-*.txt` are the user's task
prompts, one per session, committed (public work only, see below). `enclosure/archive/2026-09-dhruv-solidworks/` is a
friend's separate SolidWorks work: never edit it. `enclosure/inbox/` is an untracked
staging area for incoming material; do not commit it unless asked.

## Firmware

ESP-IDF v5.5.4, C++20, components under `firmware/components/` (`p64_gfx`, `p64_decode`,
`p64_display`, `p64_system`, `p64_playback`, `p64_storage`, `p64_content`, `p64_net`,
`p64_web`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabkury/p64](https://github.com/fabkury/p64) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->

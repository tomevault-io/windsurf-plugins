---
trigger: always_on
description: Guidance for Claude (and humans) working in this repository.
---

# CLAUDE.md

Guidance for Claude (and humans) working in this repository.

## Project goal

Build a **Sega 32X core** (Genesis/Mega Drive + 32X add-on) for the **Analogue Pocket** using
Analogue's openFPGA framework (APF). The owner has multiple physical Pockets and does all
on-device testing; Claude cannot run bitstreams on hardware, so every hardware-facing change
needs a clear test request for the owner (see `docs/hardware-testing.md`).

Planning docs live in `docs/`:

- `docs/requirements.md`: the numbered requirements and milestone plan. **Start here.**
- `docs/architecture.md`: target hardware, Pocket resource budget, memory map plan, risks.
- `docs/hardware-testing.md`: how builds get onto a Pocket, results, the regression set, releases.
- `docs/game-matrix.md`: 32X games and their status (REQ-QA-03).
- `CHANGELOG.md`: per-version changes (add an entry with each release).
- `docs/references.md`: upstream cores, datasheets and specs.

When a requirement is completed or changes, update its status in `docs/requirements.md` in
the same commit.

## Current state

M0 through M4 are done, and M6 (polish) is mostly done. Known-good builds are tagged and
published as GitHub releases from `v0.4.3`, the first public one (list in
`docs/hardware-testing.md`; v0.4.0 to v0.4.2 exist only in the private development repo).
What works on hardware:
- **Games:** most tested 32X and Genesis games play.
- **Saves:** cart SRAM/EEPROM in `s32x_save_ram.sv`, served to the APF save slot.
- **BIOS:** loaded from the SD card, with none in the bitstream; a 32X game shows a
  missing-BIOS screen if a file is absent.
- **Settings:** the full `interact.json` menu.
- **Dock:** HDMI output and player 2.
- **Look:** the "32X" icon and platform banner, and Analogue OS display modes.

Open:
- **M5:** no known open game issues (the last one, After Burner Complete's PWM weapon sounds,
  was fixed by restoring the SH-2 UBC; see `docs/m4-debug-notes.md`). `docs/game-matrix.md`
  lists the library; several games the owner has still need a first try.
- **Owner's pending hardware checks:** power-off saves, interlace, 240-line modes, SSF2, soak
  test, Pocket B (requirements marked WIP).
- **Not planned:** PAL MCLK (parked, `experiments/pal_reconfig/`), Memories (doesn't fit,
  `experiments/memories/`), Sega CD 32X games (doesn't fit, `experiments/segacd32x/`), 3-4 players
  (on request, REQ-INP-04). CI is deferred by the owner.
- **Area is tight:** synthesis estimate about 17.0k of 18.48k ALMs. Weigh the area cost of
  any new feature.

Hardware results are in `docs/test-log.md`. `docs/m4-debug-notes.md` holds the debugging
history and the simulation toolkit. The repo started as `open-fpga/core-template` v1.3.0
(commit `da3a021`).

## Repository layout

```
core.json, data.json, video.json, audio.json,   APF core definition JSON files
input.json, interact.json, variants.json         (copied into the SD card's core folder)
info.txt                                          Text shown in the Pocket's core info screen
LICENSE, NOTICE.txt                               GPL-3.0 and third-party credits/notices (both shipped in the core folder)
dist/                                             SD-card staging: icon.bin, platforms/*.json, platform images
output/bitstream.rbf_r                            Bit-reversed bitstream of the latest local build (not committed)
src/fpga/ap_core.qpf / ap_core.qsf                Quartus project (Cyclone V 5CEBA4F23C8, top = apf_top)
src/fpga/apf/                                     Analogue framework glue. Treat as vendor code; do not edit
src/fpga/core/core_top.v                          APF glue: bridge, ROM/BIOS loader, save RAM port, settings,
                                                  input, video formatter, messages, audio
src/fpga/core/s32x_system.sv                      Console: upstream gen + 32X + CART, SDRAM controller, fb_sram
src/fpga/core/fb_sram.sv                          32X framebuffers in the async SRAM
src/fpga/core/memtest.sv                          Memory self-test (tools/build.sh --memtest)
src/fpga/core/s32x_sdram_front.sv                 32X SDRAM front end (line buffer + write queue)
src/fpga/core/s32x_save_ram.sv                    Cart SRAM/EEPROM save RAM, second port on the APF bridge
src/fpga/core/pll_core.v                          PLL wrapper: MCLK 53.69, SDRAM 107.39, video 26.85 (+90°) MHz
src/fpga/core/pll/                                Generated reconfigurable PLL + reconfiguration controller (README)
src/fpga/core/s32x_msg_rom.sv                     Font/text ROM for on-screen messages (tools/gen_msg_rom.py)
src/fpga/core/rtl/S32X_MiSTer/                    Upstream submodule (pinned; patched at build time)
src/fpga/core/rtl/patches/                        Our patches to upstream, applied in order
src/fpga/core/rtl/agg23/                          agg23's MIT data_loader / sound_i2s / sync_fifo
tools/                                            build.sh, release.sh, fingerprint.sh, check_io_regs.py,
                                                  prepare_upstream.sh, reverse_bits.py, package.py,
                                                  gen_bios_mif.py (sims), gen_images.py, gen_msg_rom.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [armyofgnomes/analogue-pocket-32x](https://github.com/armyofgnomes/analogue-pocket-32x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->

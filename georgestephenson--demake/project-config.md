---
trigger: always_on
description: generates the `machine.asm` it includes. Two families have
---

# AGENTS.md — working in this repository

Guidance for coding agents (and humans) contributing to demake.
This file is the canonical project-memory file; `CLAUDE.md` is a one-line import
shim so Claude Code reads the same instructions. Keep all guidance here — never
add content to `CLAUDE.md` directly.

## What this is

A tool that **demakes modern game assets** — art, and whole games — into
something 8/16-bit-era consoles and handhelds up to the Nintendo DS could
actually run. Four demakers, sharing one engine and one proof (a real ROM, in a
real emulator, compared pixel for pixel):

| Demaker               | Docs   | State                                                                         |
| --------------------- | ------ | ----------------------------------------------------------------------------- |
| art (images)          | 03–06  | working, seventeen consoles proven on hardware                                |
| game (Demotic `.dmt`) | 14, 15 | language, interpreter, tests, preview — and playable ROMs on sixteen consoles |
| music (`arrange`)     | 16, 17 | MIDI → chip music, eighteen consoles — and every game console plays it        |
| sound (`sfx`)         | 16, 18 | WAV → chip effects, eighteen consoles — same driver, same proof               |

The four are not four tools that share a repo any more: a `.dmt` says
`music theme.mid` and `sound bounce.wav on ball hits paddle`, and `demake build`
demakes the art, the track and the effects into one cartridge — 32 KiB for a game
that fits one, and the smallest banked board its console shipped for a game that
does not (doc 14 §Sound, doc 16 §Two streams, one clock).

Every domain has the same shape, which is why they share a repo: **constrain →
fit → emit → prove it on emulated hardware**. Each reuses the layer below — a
game's sprites are demade by the image pipeline, its ROM assembled by the same
toolchain edge. The full
design lives in [`docs/`](docs/README.md); the milestone plan is
[`docs/13-roadmap.md`](docs/13-roadmap.md). **Current status: Phase 2 complete;
Phase 3 (web app) shipped** — the Phase-1 engine spine is live (the
deterministic image layer: our PNG codec, color spaces, DAC models, seeded PRNG,
math kernels; the `ConsoleSpec` schema; the tiled-and-mono conversion pipeline
with tournament + judge; the `inspect` compliance oracle). Phase 2 landed the
full proof loop for **all eight Tier 1 consoles**:

- **`prep`/`inspect` for 21 consoles** — every RGB-lattice and mono raster
  platform in doc 03 (GBC/DMG, NES, SNES, MD, SMS/GG, GBA, NDS, PCE, Neo Geo,
  WS/WSC, NGP/NGPC, VB, Pokémon Mini, Supervision, Game.com, Mega Duck) plus the
  SG-1000, through the one generic tiled fitter + mono path + the TMS9918
  Graphics II per-row two-color path (`pipeline/fit-tms.ts`). NES added
  `fixed-master` color, 16×16 attribute cells, and the shared-backdrop constraint.
- **Codegen** (`bin`/`asm`/`c`) for the `gb`, `nes`, `snes`, `sms`, `md`,
  `neogeo`, `sg1000`, `gba`, `nds`, `pce`, `ws`, `wsc`, `ngpc` and `vb`
  families, reached via an exact-path detector, a manifest sidecar, or implicit
  `prep`.
- **`--format rom`** builds bootable ROMs for GB + GBC + **Mega Duck** (RGBDS,
  the last of the three through a generated machine include rather than a
  harness of its own), NES (cc65 NROM), SMS + GG + SG-1000 (WLA-DX / Z80), SNES
  (WLA-DX / 65816, LoROM), PC Engine (WLA-DX / HuC6280, 64 KiB HuCard),
  MD/Genesis and the **Neo Geo** (GNU m68k binutils — one processor, two
  families), GBA + NDS (GNU ARM binutils), and **both WonderSwans** (NASM — the
  V30MZ is an 8086-compatible core). Two families need no assembler at all,
  because no distribution ships one for their processor: the **Virtual Boy** and
  the **Neo Geo Pocket Color** emit their display programs with `core`'s own
  V810 and TLCS-900/H encoders — the same ones `demake build` compiles a game
  with — and a third-party emulator is what keeps that honest.
  The z80/6502/65816/huc6280 assemblers are pinned source builds; the m68k and
  ARM binutils and NASM are stock distro packages (apt, main archive) since
  well-tested ones ship there — all via `pnpm toolchains`, no Docker, and no
  devkitARM/ndstool (demake packs the GBA, NDS, WonderSwan, Neo Geo Pocket and
  `.neo` cartridge headers itself).
- **Pixel-perfect emulator E2E** for every console that builds a display ROM —
  GB/GBC (SameBoy), the **Mega Duck** (SameDuck, SameBoy's own fork of that
  console, whose capturer is the Game Boy's source compiled a second time), and
  NES + SMS + GG + MD + SG-1000 + SNES + GBA + NDS + PCE + both WonderSwans +
  NGPC + Neo Geo + Virtual Boy (libretro cores via one generic
  `emu-harness/libretro/` runner) — all marching through the same shared
  extensive image battery (`packages/cli/test/_emu-battery.ts`).

Phase 5 then opened Tier 2 with the **PC Engine** and the **WonderSwan Color**,
both riding that same loop end to end (`wla-huc6280` on the existing WLA-DX
build and beetle-pce-fast; NASM and beetle-wswan). Doc 13 §Phase 5 records what
blocks each remaining Tier 2 console.

**And the mono WonderSwan demakes art now**, which took a fit path rather than a
toolchain. That console's palette has a level of indirection nothing else in the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [georgestephenson/demake](https://github.com/georgestephenson/demake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

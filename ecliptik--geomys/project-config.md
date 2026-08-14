---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Geomys is a Gopher browser for classic Macintosh (68000/Macintosh Plus) written in C, targeting System 6.0.8 with MacTCP. Implements RFC 1436 (Gopher) and RFC 4266 (Gopher URI scheme).

## Target Platform Constraints

- Motorola 68000 CPU only (no PowerPC, no 68020+ instructions)
- 4 MiB RAM on Macintosh Plus
- System 6.0.8 primary target
- MacTCP for networking
- Monochrome only for MVP
- Latency and responsiveness are critical priorities

### Multi-Window Memory Constraints (System 7 Color)

On System 7 with Color QuickDraw, a shared 8-bit GWorld offscreen buffer (~300KB at 640x480) is used for flicker-free rendering. Combined with per-page item arrays (298 bytes × item count with Gopher+), memory is the primary constraint for multi-window support:

- **System 6 (monochrome)**: 1-bit offscreen (~22KB). Memory is rarely an issue.
- **System 7 (color)**: 8-bit shared GWorld (~300KB) + per-window item arrays.
- **GopherItem size**: ~298 bytes each (display[100] + selector[128] + host[64] + Gopher+ fields). A large directory (1500+ items) uses ~435KB.
- **Practical window limits**: 2-3 windows with large directories on 4MB. Item array growth (128→256→512→1024→2000) fails silently when heap is exhausted - second/third windows may show fewer items than the first.
- Allocation failures show as "Connection failed" or truncated item lists - the item array `NewPtr` returns NULL and stops adding items at the current capacity boundary (256, 512, 1024).

### Tuning for More Memory

The SIZE resource controls how much memory MultiFinder/System 7 gives Geomys. Default values by preset:

| Preset | Preferred | Minimum | Max Windows |
|--------|-----------|---------|-------------|
| Minimal | 512KB | 256KB | 1 |
| Lite | 1024KB | 768KB | 2 |
| Full | 2560KB | 1536KB | 3 |

On machines with more RAM (8MB, 32MB+), increase the SIZE resource clamp in `scripts/build.sh` (lines 216-221) to give Geomys a larger heap partition:

```bash
# Example: raise full preset to 8MB preferred / 4MB minimum
local max_pref=8192
local max_min=4096
```

Users can also adjust memory after building via Finder's "Get Info" on the Geomys application - change "Application Memory Size" to the desired value. This does not require rebuilding.

## Build System

- Cross-compile on Linux using [Retro68](https://github.com/autc04/Retro68) toolchain
- Toolchain built from source, installed at `Retro68-build/toolchain/` (in-repo, gitignored)
- Source cloned at `Retro68/` (in-repo, gitignored)
- Build: `./scripts/build.sh` (re-use build.sh and release.sh patterns from Flynn)
- CMake flag: `-m68000` for Mac Plus compatibility
- Target artifacts: 800K `.dsk` floppy images and `.hqx` (BinHex) compressed binaries
- Retro68 API quirks vs classic Toolbox: `qd.thePort` not `thePort`, `GetMenuHandle` not `GetMHandle`, `AppendResMenu` not `AddResMenu`, `LMGetApplLimit()` not `GetApplLimit`

### Debug Build Flag

`--debug` enables `GEOMYS_DEBUG` which adds diagnostic info to the status bar (item counts, timing). Example: `./scripts/build.sh --preset minimal --debug`

### Feature Flags

Per-feature CMake options are defined in `CMakeLists.txt` and surfaced via `--flag`/`--no-flag` in `scripts/build.sh`. See `docs/BUILD.md` for the full table. `GEOMYS_UTF8` (auto-detected UTF-8 decoding for directory titles and text content, transcoding to Mac Roman with a CP437 fallback) is ON in full and lite presets, OFF in minimal (mirrors `GEOMYS_CP437`).

### Releases & Artifact Verification

`scripts/release.sh vX.Y.Z` builds all 3 presets (`build.sh --clean --preset {full,lite,minimal}` → 9 artifacts: `.dsk`/`.hqx`/`.sit`) then tries to publish to Forgejo/GitHub; with no `FORGEJO_TOKEN`/`gh auth` it skips publishing and leaves the artifacts in `build/` (ready for Macintosh Garden upload). The build stamps the version only if tag `vX.Y.Z` is at **HEAD** — bump `CMakeLists.txt`, the resource file (`geomys.r` + its `'vers'` resources), and `docs/About Geomys`, finalize CHANGELOG, then commit + tag before building.

Verify artifacts host-side (no Mac): `.dsk` via `hmount` (valid HFS + app); `.hqx` via `hexbin` (decoded `.bin` differs only in MacBinary header dates — resource fork is byte-identical); `.sit` via `macunpack -f` (resource fork byte-identical). To confirm a `.sit` opens in real StuffIt, test in **Basilisk/System 7** (System 6 has no Expander). Details in the `reference_release_and_artifact_verification` and `reference_sit_stuffit_basilisk_test` memories.

## Testing

Emulator infrastructure lives in `/home/claude/emulators/`. See its docs for full details:
- `docs/TESTING.md` - build-deploy-test workflows for both emulators
- `docs/SNOW.md` - Snow emulator config, networking, keyboard
- `docs/SNOW-GUI-AUTOMATION.md` - X11/XTEST coordinate system and automation techniques

**IMPORTANT: Do NOT launch Snow, deploy to disk images, or run any automated testing unless the user explicitly asks. All testing and QA is done by the human. Only build and deploy when asked.**

### Emulator: Snow (System 6 - Primary)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecliptik/geomys](https://github.com/ecliptik/geomys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

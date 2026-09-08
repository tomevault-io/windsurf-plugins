---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

This file is a **map**, not the manual. The technical content lives in `docs/` and the per-crate `README.md`s; this file points at the right page for whatever you're touching, plus the rules of engagement that apply across the whole repo.

Keep it that way when you edit it. A table row here is a one-line "what this covers" plus the link - if you find yourself writing a spec into a cell, the spec belongs on the linked page instead.

## Project mission

Two coordinated tracks under one repo (`-re` = reverse-engineering, in both senses):

1. **Asset preservation + format docs.** Extract every asset on the disc, document every format with Ghidra-traced provenance, build round-trip parsers.
2. **Engine reimplementation.** From-scratch Rust port - render via wgpu, audio via the XA + VAB decoders, optional WASM target. End-user model: ship the engine, user supplies the disc image, engine extracts and runs.

The public framing is **a playable port and modding hub standing on Ghidra-traced reverse engineering** - fresh Rust from format docs + decompiled-C reference (ScummVM / OpenRCT2 model), not a decompilation project and not a static recompilation of `SCUS_942.54`. Don't call it "clean-room" in committed prose (the same people read the dumps and write the Rust); the enforced boundary is in [`docs/subsystems/engine.md`](docs/subsystems/engine.md).

**"Port" does not mean 1:1.** Retail behaviour is the measured ground truth - the traced dumps and parity oracles pin it exactly, and a retail-faithful mode stays available and testable - but the port is not bound by it. New features, mechanics, rendering and audio are in scope; enhancements ship enabled by default where they are clearly the better experience, with retail one toggle away.

The enhancement layer today: dynamic lighting (`--dynamic-lighting`, pixel-identical when off), the camera-occlusion fade (see-through walls around the player, default-on in `play-window`, `--no-occlusion-fade` / `F4` disables), precise free-angle movement (`options::precise_movement`), the debug orbit camera, and VR - all toggles that leave the faithful mode bit-identical when off. Per-knob current defaults live in [`docs/subsystems/engine.md`](docs/subsystems/engine.md#fidelity-and-enhancements); a knob still defaulting to retail marks an enhanced side that is maturing, not a policy of restraint.

The render path splits in two, and the halves point opposite ways. **Shading defaults to retail**: the game's textured / colour mesh paths draw the TMD's baked colour word through the GTE depth cue and apply no light source at all. **Rasterisation defaults to clean**: `Renderer::set_psx_mode` is opt-in and gates vertex jitter + 15-bit dither only. Affine UVs are not gated - they are unconditional, and they are the faithful behaviour.

The shading half now holds on **both** hosts. The synthetic Lambert survives in exactly two viewer aids - `MESH_SHADER_SRC` (the asset-viewer's bare-geometry preview) and the site's deliberately-lit bestiary preview - neither of which is a claim about retail. `site/js/webgl-shaders.js` once applied that Lambert on *both* its paths on every 3D page; both are retail now, the page uploads per-vertex packet colour, and no light uniform remains. The trap that kept it alive: an unbound colour attribute defaults to white, and white is `texel * 255/128`, so a missing colour stream reads as "too bright", not as "unlit". Tracked in [`docs/tooling/host-drift.md`](docs/tooling/host-drift.md).

Modding (`crates/patcher`) and translation are designed, shipped tracks, not side-effects - and what the patcher proves out against retail (randomizer logic, softlock fixes, tuning sliders) is expected to graduate into engine toggles. What the project still is *not*: a static recompile, and never a build that silently loses the retail-faithful mode.

**Sony IP (executable, ROM contents, asset bytes) is NEVER committed.** `extracted/` is gitignored, disc-dependent tests skip when `LEGAIA_DISC_BIN` is unset, no decompressed Sony bytes (text strings, sample data, decompiled-C dumps with literal data) get checked in. CI runs without disc data.

## Repository map

The committed docs are organised topic-first under `docs/` - public-facing technical reference, no progress tracker / session log / status tables. Operational state lives in git log + the agent-only memory directory at `~/.claude/projects/-home-mikunpc-Documents-repos-legend-of-legaia-re/memory/`.

### Top level

- [`README.md`](README.md) - public project overview, build instructions, license.
- [`docs/overview.md`](docs/overview.md) - elevator pitch + how the layers stack from disc to sub-asset.
- [`docs/guides/`](docs/guides/getting-started.md) - task-oriented user guides: getting-started, extracting-assets, playing-and-viewing, modding-and-translation.

### Formats - [`docs/formats/`](docs/formats/overview.md)

Per-format byte-level specs with Ghidra-traced provenance. Read the relevant page before writing a parser; don't guess from the data.

| Doc | Covers |
|---|---|
| [`overview.md`](docs/formats/overview.md) | Index page; confidence levels (Confirmed / Inferred / Unknown); format families. |
| **Disc + container layer** | |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndrewAltimit/legend-of-legaia-re](https://github.com/AndrewAltimit/legend-of-legaia-re) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

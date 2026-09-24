---
trigger: always_on
description: These instructions apply to every change in this repository. They are the
---

# DKC1Recomp Project Rules

These instructions apply to every change in this repository. They are the
default operating rules for Codex and other coding agents working on DKC1Recomp.

## Project objective

Produce a playable, deterministic native recompilation of DKC1 with optional
widescreen presentation while preserving the cartridge game's behavior and the
native 256x224 image. A visually wider frame is not a valid result if it damages
the native image, object lifecycle, collision, exits, bosses, timing, or save
state continuity.

## Required widescreen model

Treat every widescreen issue as three independent domains until evidence proves
otherwise:

1. **Presentation:** host framebuffer, crop, margins, layer composition, masks.
2. **Streaming and activation:** VRAM tile rows/columns, ring buffers, OAM,
   object scanner windows, allocation, lifecycle.
3. **Gameplay logic:** collision, pickups, exits, movement limits, boss arenas,
   scripted state machines.

Do not repair one domain by silently changing another. A visual-only correction
is incomplete when interaction or gameplay coordinates still disagree.

## Non-negotiable defaults

- Stock cartridge behavior is the default for every unproven scene and layout.
- Widen presentation globally only where the host can do so without mutating
  cartridge state.
- Widen cartridge streaming, activation, or logic only after that exact
  subsystem and layout have passed the promotion gates below.
- Unsupported presentation states fail closed to stock output or black margins.
  Never repeat, wrap, or invent unverified side art.
- The original 256-pixel center is an oracle. It must remain pixel-exact unless
  a deliberate center correction is separately documented and approved.
- Keep the experimental cartridge initializer and row-stream widening disabled
  by default. `DKC1_ENABLE_EXPERIMENTAL_CARTRIDGE_WIDENING=1` is a research
  switch, not a release configuration.
- Reaching a shared DKC initializer, streamer, or renderer is not proof that a
  particular room or tilemap supports widened cartridge writes.
- Prefer explicit, evidence-backed capabilities over scene denylists. A scene
  tuple may be used as a narrow containment guard, but it is not the long-term
  architecture.

## Authoritative source and decompile library

Use `reference/` for all decompile, disassembly, IDA, pseudocode, RAM-map, and
historical widescreen research. Check it before searching external paths or
starting a new decompilation.

- `reference/disassembly/` is the primary byte-exact DKC1 source oracle. It
  contains the labeled disassembly, RAM and define maps, mechanical pseudocode,
  lossless listing, instruction index, seeded IDA database, curated rename map,
  reverse-engineering notes, and byte-identical assembly inputs.
- `reference/legacy-widescreen/` is historical evidence from the retired
  emulator/asar effort. Use its worklogs and tools to recognize bug classes,
  but never treat its patched ROM behavior as stock or copy its ROM-patch
  architecture into the recomp without new evidence.
- `reference/dkc-recomp-seed/` is the older bring-up seed. Use it for historical
  comparison only.
- Start an address or symbol investigation with
  `python tools/atlas.py <address|wram:address|name:term>`. The atlas joins the
  reference disassembly, pseudocode, IDA annotations, recomp variant, dispatch
  contracts, WRAM labels, and known issues.
- Ground every symbolic claim in the clean-ROM bytes or byte-exact reference.
  Names and decompiler output are navigation aids, not proof of semantics.
- Treat `reference/` as read-only. Most of its contents are intentionally
  ignored because they are large, generated, externally versioned, or contain
  build inputs that must not be committed. Fixes land in `runner/`, `recomp/`,
  `tools/`, `recipes/`, `contracts/`, or documentation as appropriate.
- Never hand-edit `generated/`. Recompilation-correctness fixes belong in
  `recomp/*.cfg` followed by regeneration; presentation fixes belong in the
  host/adapters or the pinned engine when explicitly scoped.

Read `reference/README.md` before moving, regenerating, or updating any source
oracle. Preserve the nested disassembly repository and its history.

## Before changing widescreen code

1. Read `docs/WIDESCREEN.md`, `docs/WIDESCREEN_HANDOFF.md`,
   `docs/WIDESCREEN_DEBUG_TOOLS.md`, and `docs/KNOWN_ISSUES.json` for the
   affected subsystem.
2. Record the clean ROM hash, executable/build identity, state hash, level,
   entrance, mode, aspect ratio, frame, and exact input schedule.
3. Preserve supplied save states outside the normal slot directory. Never
   overwrite the only tester reproduction.
4. Capture both the final visible window and raw WRAM/VRAM/CGRAM/OAM/PPU or
   isolated BG/OBJ evidence. An internal render target is not the sole visual
   oracle.
5. Establish one-variable stock-versus-candidate A/B evidence before editing.
6. Use one live automation owner at a time. Do not race controller, bridge, or
   save-state operations from multiple processes or agents.

## Save-state rule

Every state-based visual or lifecycle report requires two branches:

- **Exact-state branch:** reproduce and diagnose the machine state the tester
  supplied.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elliotttate/DKC1Recomp](https://github.com/elliotttate/DKC1Recomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

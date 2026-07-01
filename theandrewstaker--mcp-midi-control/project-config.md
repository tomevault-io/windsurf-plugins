---
trigger: always_on
description: Read by Claude Code at the start of every session.
---

# MCP MIDI Control, Claude Code Context

Read by Claude Code at the start of every session.

---

## Project Purpose
A local MCP server that lets Claude (or any MCP host) control MIDI gear
over USB by conversation. It is an opinionated project: a set of strict
rules for controlling music gear (display-first units, no silent
saves/overwrites, acknowledged writes, tempo-first, read-before-write)
that apply consistently to every device. Generic-MIDI primitives reach
any USB MIDI device; a first-class tier gets full preset/patch authoring.
The deepest current support is for the Fractal AM4, Axe-Fx II, and ASM
Hydrasynth, with the modern Fractal family (Axe-Fx III / FM3 / FM9) and the
original Axe-Fx Standard/Ultra (gen-1, set + parameter read) in community beta. Consistency across
devices is a core value, and adding new gear (Line 6 Helix and other
popular modelers, instruments, and synthesizers are the wanted targets)
is a descriptor, not new tools.

## Current Phase
**Status:** pre-release. AM4 + Hydrasynth functional; Axe-Fx II functional; the modern Fractal family (Axe-Fx III / FM3 / FM9) in community beta via one shared gen-3 codec factory — the FM3's core surface (USB-serial transport, reads, continuous param writes, bypass, scenes, preset switching) is hardware-CONFIRMED end-to-end through this server's own probes by a 2026-06-12 community field test, and set-by-name discrete param writes are FM3-hardware-confirmed via a 2026-06-10 collaborator session (frames byte-identical to our encoder, sent from the tester's rig); set_block, save_preset, and the Windows serial-driver path still await on-device confirmation. The **FM9** is now also community-confirmed for the read + continuous-write path: a 2026-06-17 owner test (fw 11.0 / macOS) round-tripped `get_param` + continuous `set_param` on hardware through this server (acked, values confirmed on the FM9-Editor display), plus channel-specific reads and alias resolution; discrete set-by-name, `save_preset`, `set_block`, and the new live grid read (fn=0x01 sub=0x2E) stay community-beta on the FM9. The **Axe-Fx III** (the gen-3 byte-identity anchor) is now also hardware-confirmed end-to-end for the first time: the same 2026-06-17 owner test ran `set_param` (amp gain, channel A) with a device echo and a `get_param` read-back matching the front panel (same beta carve-outs as the FM9). A 2026-06-18 community roundtrip then exercised SET→GET across the entire FM9 and III catalogs on hardware: it confirmed the read + continuous-write paths catalog-wide and surfaced one fixed catalog bug — FM9 enum/type selectors were routing as continuous floats instead of discrete ordinals (~351 params corrected, gated on the FM9's own editor-cache enum data, so only the FM9 changed and the III byte-identity is intact; III/FM3/VP4 need a device-synced editor cache to get the same, the copies on hand are unsynced stubs with no enum vocabulary). That roundtrip is device-behavior evidence from an independent rig; this server's own discrete write frame stays community-beta. The original Axe-Fx Standard/Ultra (gen-1, model 0x01) in community beta, with parameter set + read decoded from the published gen-1 SysEx spec. See ROADMAP.md.

Start a session by reading the maintainer's private operational notes
(gitignored, not in the public tree): the current-state doc names the
phase, the single next action, and recent findings, with per-device
shards for device-targeted work. A private per-device hardware-task list
queues the hardware actions the maintainer owes; if a pending task gates
the work you are about to do, flag it before proceeding.

The maintainer's private operational scratch (gitignored: state,
hardware tasks, session log, backlog, decisions log, test plans) lives
outside the committed tree. Committed `docs/` files cover MCP-server
architecture and contract (ARCHITECTURE.md, BLOCK-PARAMS.md,
PROJECT-VISION.md, SAFE-EDIT-WORKFLOW.md, etc.). Protocol RE (per-device
SYSEX-MAP, capture guides, Ghidra scripts, encoding cookbook) lives in
the [`fractal-midi`](https://github.com/TheAndrewStaker/fractal-midi)
codec package under `packages/fractal-midi/docs/`.

## Shipping bar: evidence, not hardware (read this before deferring anything)

**The bar for shipping a capability is EVIDENCE, not a device key-press.** If the
wire/decode logic is derived from evidence we can actually check, SHIP IT and mark
it untested — do not withhold or discount it for lack of hardware verification.
Withholding likely-correct capability is the failure mode this project keeps hitting;
it holds us back and confuses what the product can actually do.

Judge work on **two independent axes**, and never collapse them into one "unverified":

1. **Evidence strength** — is the logic grounded in something checkable?
   - STRONG: byte-exact against a real capture we hold; self-validating (the device's
     own CRC/checksum gates it, e.g. the gen-3 `.syx` CRC); byte-identical round-trip;
     cross-validated against a reference oracle; derived from a published spec read
     byte-for-byte. → **Ship it. Mark "untested / community-beta." It is DONE pending a
     confirmation key-press, not "not done."**
   - WEAK: a guess with no way to catch a wrong answer — an unvalidated join, an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAndrewStaker/mcp-midi-control](https://github.com/TheAndrewStaker/mcp-midi-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

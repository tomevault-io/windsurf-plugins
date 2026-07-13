---
trigger: always_on
description: This file is the master plan and working guide for agents (Claude Code / Opus)
---

# CLAUDE.md — sndj

This file is the master plan and working guide for agents (Claude Code / Opus)
building **sndj**. It is written before any code exists; as milestones land,
sections here graduate into DESIGN.md (the contract), MANUAL.md (the player's
guide), SAVEFORMAT.md, and HARDWARE.md, exactly as in the sibling repos.
Until then, **this document is the contract.** Read the relevant section before
making design decisions; decisions marked ⚖ SETTLED are not to be re-litigated.

---

## 1. Project identity

**sndj** — an LSDJ-inspired music tracker for the **Super Nintendo /
Super Famicom**, written in **65816 + SPC700 assembly**. It is the third
sibling of:

- **[smsggdj](https://github.com/little-scale/smsggdj)** — Sega Master System / Game Gear (Z80, SN76489 PSG)
- **[genmddj](https://github.com/little-scale/genmddj)** — Sega Mega Drive / Genesis / Nomad (68000 + Z80, YM2612 + SN76489)

The sibling contract — what carries over **verbatim in spirit** unless the SNES
hardware forces otherwise:

1. **Data model**: notes live in PHRASEs (16 rows = 1/16th notes), phrases live
   in CHAINs, chains are arranged per-track in the SONG grid. Tracks map 1:1 to
   hardware voices. Tracks do not own instruments/phrases/chains — they play
   *out of* shared pools.
2. **Control grammar** (⚖ SETTLED, from DESIGN.md §3 of smsggdj): two modifier
   buttons; *the button already held when the other arrives selects the
   action*; never introduce simultaneous-press timing windows. Item-level
   modifier = insert/edit/nudge/paste/audition; project-level modifier =
   screen navigation and transport.
3. **Screen set**: SONG / CHAIN / PHRASE / INSTR / TABLE / WAVE / GROOVE /
   ECHO / FILES / PROJECT / OPTIONS / LIVE, arranged on a 2-D screen map
   navigated by (screen-modifier held) + d-pad. sndj adds SNES-specific
   screens (KIT, FIR, MIDI — §8).
4. **Command set**: the shared A–Z single-letter command language, one executor
   shared by phrase and table columns, `cmd_chars`/`cmd_order` id↔letter↔rank
   mapping. SNES-specific commands extend the set; shared letters keep shared
   semantics (§10).
5. **Grooves are the tempo** (DESIGN §9 in smsggdj): TMPO is a live readout
   derived from the active groove, not an independent clock.
6. **Sync family**: `OFF / OUT / PULSE / IN / IN24`, numbered like genmddj;
   OUT/IN are a one-wire D0 toggle per row (lock two machines at any tempo), IN24 is
   24 PPQN for the Ableton Link ESP32 bridge. Cross-sibling sync is a design
   goal: a Mega Drive and a SNES on one cable must lock (§12).
7. **Engine split** (genmddj model): the main CPU owns *everything* — song
   data, editor, UI, per-tick sequencer. The sound CPU is a pure chip servant
   holding no song state. Per tick, the main CPU computes desired chip state,
   diffs against a locally held shadow, and ships only the changes as a small
   **Sound Control Block (SCB)** (§3).
8. **Ecosystem**: browser-first, zero-toolchain tools for musicians (ROM
   patchers for samples/palette/font/presets, a save/song manager, the
   Ableton/MIDI/MML converter), plus Python CLI mirrors of each; one shared,
   node-testable JS library (the `smdj4.js` pattern) that every tool imports
   (§17).
9. **Build hygiene**: `make` emits version + git-hash-stamped dev copies;
   `make dist` emits version-only release copies; boot splash shows the build
   stamp to catch stale flashes; per-version CHANGELOG.md; MIT license.

Naming (⚖ SETTLED unless Seb objects):

| Thing              | Name                        |
|--------------------|-----------------------------|
| Repo / project     | `sndj`                    |
| ROM                | `build/sndj.sfc`          |
| Song file          | `.sndj`                     |
| Save format magic  | `SNDJ1` (family: SMDJ3/4)   |
| SPC driver blob    | `build/driver.spc700.bin`   |
| Reference JS lib   | `user-tools/sndj.js`        |
| ESP32 bridge repo  | `sndj-link-esp32`         |

Voices (8): `V1`–`V8`, all hosted by the S-DSP. Unlike the siblings there is
no chip heterogeneity — every voice is a full citizen (sample, kit, wavetable,
or noise). Heterogeneity on the SNES comes from *instrument type*, not from
which column you're in. This is the single biggest data-model relaxation
versus the siblings and should be embraced, not fought.

---

## 2. The instrument: what makes the SNES sound special

The S-DSP is not a synthesizer chip with a sampler bolted on — it is a
**sampler with a mixing console, a modulation bus, and a room built in**.
sndj should be designed around the five things only this chip does:

1. **BRR + Gaussian interpolation = the SNES timbre.** All samples are
   4-bit-nibble BRR blocks (9 bytes → 16 samples, four prediction filters),
   played back through a 4-tap Gaussian interpolator that rolls off highs.
   The grit of BRR quantisation under the softness of the Gaussian filter *is*
   the sound (Super Metroid, DKC, Secret of Mana). Consequence for tooling:
   the browser patcher must audition samples through a **bit-exact BRR
   encode→decode→Gaussian→32 kHz** path so what you hear is what the console
   plays, and the encoder must offer **treble pre-emphasis** to pre-compensate
   the Gaussian rolloff (the classic BRRtools trick).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [little-scale/sndj](https://github.com/little-scale/sndj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->

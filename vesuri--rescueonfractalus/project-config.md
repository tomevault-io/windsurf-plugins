---
trigger: always_on
description: Reimplementing the 1985 Atari 8-bit game *Rescue on Fractalus!* on the Amiga from a
---

# Rescue on Fractalus! — Atari 8-bit → Amiga port

Reimplementing the 1985 Atari 8-bit game *Rescue on Fractalus!* on the Amiga from a
binary only (`rof.xex`, no source). Pipeline: decompile (Ghidra) → transliterate 6502
→ C → abstract hardware → platform backends (SDL on macOS for dev, Amiga A500 as
the real target). **Faithful 1:1 port** — parity before improvements; validate against
the Atari 6502 code + `atari800`, NOT PlatformSDL (SDL is an approximation).

## Phase vocabulary (use these names everywhere)

The boot→flight sequence has 7 canonical scenes (user-approved). Code ids are `SCENE_*`.

| # | Scene | What it is |
|---|---|---|
| 1 | **Logo** | Lucasfilm Games boot logo |
| 2 | **Station** | Space-station cinematic (stars scroll, station animates). `station_init $195D` |
| 3 | **Standby** | Cockpit + "RESCUE ON FRACTALUS!" title + LEVEL doors, awaiting START |
| 3b | **Title Screen** | Attract/level-select/results card: big mode-7 "RESCUE ON FRACTALUS!" + mode-6 copyright / STARTING LEVEL / RANKING LEVEL / LAST SCORE / HIGH SCORE on black, text pens cycle. Shown on Standby idle (attract timeout), on **SELECT or joystick-up from the initial Standby** (both faithful — same dispatch branch, measured 2026-08-03), or after a crash. DL `$5A82`, charset `$0400`, screen RAM `$365B`. (Was "Scoreboard".) |
| 4 | **Doors** | Hangar doors opening (start of launch) |
| 5 | **Tunnel** | Tunnel/descent |
| 6 | **Planet** | Planet approach |
| 7 | **Flight** | In-game terrain rendering (gameplay) |

The Amiga app's main class is `RescueOnFractalus` (flight is a continuation of it, not a
separate scene). Atari entry is `game_entry $3CDE`; main blob `$3CDE–$B7FF`.

## Reference docs — READ ON DEMAND (this file stays small on purpose)

Hard-won detail lives in `docs/`, not here. **Read the relevant one BEFORE working in its area**;
these are the derived facts we must not re-derive or contradict.

| Doc | Read it when |
|---|---|
| `docs/scene-composition.md` | Working on any scene's DL / screen modes / PMG / copper list / windscreen frame |
| `docs/instruments.md` | Touching the cockpit HUD, a named instrument, enemies or terrain objects |
| `docs/controls.md` | Touching input, the keyboard/console path, Standby SELECT, or BREAK/restart |
| `docs/perf-budget.md` | Quoting, sizing or judging ANY performance number; measuring an asm twin |
| `docs/m68k-optimisation.md` | Optimising a hot function or writing an asm twin (68000 rules) |
| `docs/headless-fsuae.md` | Writing a probe, driving FS-UAE headlessly, or suspecting a stale build |
| `docs/transpiler.md` | Working on `tools/transpile.py`, or when generated-code shape surprises you |
| `docs/asm-migration-plan.md` | Any asm twin work — the per-phase design record |
| `docs/flight-perf-log.md` | The perf investigation archive: what was tried, what it measured, what closed |
| `docs/sfx-events.md` | Audio: the 33 SFX events, the voice engine, captured POKEY streams — **and the three music players** (Standby theme `$70F9`, the two jingles `$7253`, vs non-musical `station_audio`), the poly-distortion pitch rule, and `tools/export_midi.py` |
| `docs/rename.md` | A function's name contradicts its behaviour (append to it — see conventions) |
| `docs/high-score-initials.md` | Why the high-score INITIALS entry (`name_entry_loop`) was DEAD — a DISK save-state check whose two sector reads are NOPped out of `rof.xex`. The derivation, not current behaviour; and `make NAME_ENTRY=1` to reach the screen |
| `docs/high-score-restore-plan.md` | The high-score table + initials entry as BUILT (2026-08-18): the block (`src/rof_hiscore.c`; factory bytes now come from `rof.rom`), restored SIO calls at `$5D86`/`$5D9D`, `RoF.hi` persistence, the entry screen's DL `$5E2E` + shared text decoder, keyboard map, and §4 open items |
| `docs/rom-v50-diff.md` | Anything involving `rof.rom` — the 64 KB **v5.0 CARTRIDGE**. ⚠ 5.0 is the CONSOLE (XEGS) version, 4.1 the COMPUTER version we implement: adopt its FIXES, not its console input. Carries the bank/runtime map, the xex↔cart address map + how to re-derive it, the **authoritative Logo→Station→Standby boot chain** (4.1's is entangled with the `.xex` loader), and the full difference inventory |
| `docs/whdload-slave.md` | Touching the WHDLoad install (`whdload/`) — the slave, the install package, or its memory sizes. Carries WHY it is a `kick13.s` kickemu rather than a plain loader (the port needs the OS), the measured RAM budget + how to re-tune it, and the Install-Template edits |
| `docs/asset-extraction.md` | Historical sparse-image work (now superseded for shipping data), **anything `__chip`**, or judging `out/RoF`'s size. Carries the COVERAGE method, A/B traps, and §6 size ledger — incl. **`.MEMF_CHIP` is a BSS hunk, so a `__chip` static initialiser is silently discarded** |
| `docs/copyright-data-extraction-plan.md` | Externalising original game data: the agreed XEGS-ROM source, BSS/default versus standalone build contract, WHDLoad loading, asset retirement scope, and verification gates |
| `docs/logo-station-plan.md` | Scenes 1 (Logo) + 2 (Station): screen composition, the routines, and the Amiga port plan. ⚠ Also carries the correction that boot `INITAD $5000` is the LOGO, not `stage_5000` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vesuri/rescueonfractalus](https://github.com/Vesuri/rescueonfractalus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

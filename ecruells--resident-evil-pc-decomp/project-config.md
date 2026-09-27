---
trigger: always_on
description: A decompilation of the 1997 Resident Evil 1 PC release (the USA build, as shipped
---

# Resident Evil 1 for PC Decompilation Project

## What this is

A decompilation of the 1997 Resident Evil 1 PC release (the USA build, as shipped
by GOG — same binary as the 1997 retail disc). The code is reconstructed as a
Win32 C++ project that reproduces the original binary's behaviour, not a
redesign: original function and global addresses stay in comments, and the
original's quirks are preserved rather than cleaned up.

## Current state

- **USA port: function-complete and playable.** 1800 of 1801 in-scope game
  functions are implemented; the single remainder
  (`ent_setanim_walkto_helper`, `0x0040c3d0`) is documented. The Release build
  has been play-tested end to end by the user. Remaining work is bug fixes and
  new features — not finding missing functions.
- **JPN (Biohazard, MediaKite release): partial.** The Japanese text encoding,
  the message / item-name / item-description / font tables and the save/load
  screen strings are ported, generated into `src/game/JpnTextTables.cpp` and
  selected at runtime with `config.ini [Assets] Version=JPN`. Everything else
  that release has and the USA build does not is **not ported yet**.
- **Linux port: playable.** A native 32-bit binary (`-m32`) shares every game
  translation unit and swaps the platform layer plus the renderer backend
  (SDL2 + OpenGL 3.3 core, ffmpeg for FMV). Verified running on Ubuntu and
  CachyOS, keyboard and gamepad. See `docs/LINUX_PORT.md`.

## Project goal

The USA release is no longer the end of the project. The goal now is to **add
features from other RE1 releases** on top of this port, starting with the
Japanese MediaKite version. Extend behaviour; keep the existing USA behaviour
as the default so the two do not silently diverge.

## Ghidra programs

Two programs are available in the Ghidra project. Both are reverse-engineering
sources for this repo:

| Program | What it is | Use it for |
|---|---|---|
| `ResidentEvil.exe` | USA PC release (1997 / GOG) | the primary source — every address, function and global already in this repo comes from it |
| `Biohazard.exe` | Japanese MediaKite PC release | the features, text and data the USA build does not have; the JPN tables in `src/game/JpnTextTables.cpp` were generated from it |

- Query code through the MCP Ghidra server. **If the server is unavailable,
  stop and ask the user for help — do not guess code from memory.**
- Addresses differ between the two programs. If an address does not match what
  you expect, ask the user which program is currently loaded.
- When you port something from `Biohazard.exe`, name the program in the comment
  so it is not mistaken for a USA address, e.g. `// 0x004912C0 (Biohazard.exe)`.
- The entry point in the USA program is `main` at `0x00441350`.

## Reverse engineering rules

- Comment the original address of every function you rewrite and every global
  you declare.
- Name unnamed functions, globals and locals — and rename them in the Ghidra
  project too, so the next session sees the same names.
- Implement the dependencies of what you port. Do not stub or skip functions;
  this project keeps as much of the original code as possible.
- **Global placement.** Before defining a new global, read
  `docs/MEMORY_LAYOUT.md` and check its original address:
  - `0x00be41e0..0x00be9620` (game-init wipe range) → the global must also be
    cleared by `ResetGameStateBlock()` in `src/game/GameStart.cpp`. The old
    `.gwipe` / `.sched` / `.items` linker sections are **gone**; every global is
    now an ordinary definition and the wipe is an explicit per-name clear.
  - `0x00be9620..0x00be9a3c` (bio card / save block) → a `BioCardLayout` field.
  - Never rely on linker adjacency for a range operation (`memclr`/`memcpy`/
    pointer walk across two different globals) or for past-the-end addressing —
    model the range explicitly. Getting this wrong corrupts memory silently.
  - The "Mechanism" sections of `docs/MEMORY_LAYOUT.md` predate the section
    removal; treat them as history, not as instructions.
- If you cannot test a change yourself (i.e. playing the game), do not assume it
  is solved — ask the user to test and tell you the result.

## Tools (`tools/`)

`tools/` holds decompilation tooling and the verifiers whose artifacts live
there; `tests/` holds the test harness (`check_platform_boundary.py`,
`compile_linux.sh`, `rgba_to_png.py`). Keep that split.

Most tools read the original binaries (`assets/ResidentEvil.exe`,
`assets/Biohazard.exe`) or the shipped assets (`assets/USA|JPN/...`) and print
decoded data — they are how a table is confirmed against the original instead
of guessed. Run them from the repo root.

**Extraction from the original binaries**
- `decode_re1.py` — decode the game's custom text encoding (`PrintFormattedText` / `PrintText8x14` strings).
- `decode_msg_table.py` — decode the global message table from `ResidentEvil.exe` for comparison with the `STR()` sources in `Globals.cpp`.
- `extract_global_messages.py`, `extract_item_descriptions.py` (table at `0x004C6160`), `extract_string_table.py` — pull specific tables out of the original binary.
- `mine_effect_tables.py`, `gen_effect_c_tables.py` — mine the billboard-effect tables and emit them as C initializer lists for `EffectSystem.cpp`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecruells/resident-evil-pc-decomp](https://github.com/ecruells/resident-evil-pc-decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

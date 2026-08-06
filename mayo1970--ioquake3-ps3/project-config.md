---
trigger: always_on
description: > Agent guidance for this repo. Dense by design; read it fully before editing.
---

# CLAUDE.md — ioquake3-PS3 port (PSL1GHT / devkitPro)

> Agent guidance for this repo. Dense by design; read it fully before editing.
> End users want [README.md](README.md).
>
> **This file is the master. [AGENTS.md](AGENTS.md) is a verbatim mirror of it**
> (Codex and other tools read AGENTS.md). Edit CLAUDE.md, then copy it over
> AGENTS.md so the two never drift. If they ever disagree, CLAUDE.md wins.

---

## Maintaining this file (read first)

This doc rotted into an 82 KB dated changelog once; keep it from happening again.

**Add an entry here only if ALL of these are true:**
1. It is a durable invariant or constraint, not a one-time event.
2. Violating it breaks the build, boot, or gameplay.
3. It is **not** obvious from reading the code.

The test: *"Would an agent re-introduce this bug if this line were deleted?"*
If no, it does not belong here.

**Do NOT add:** dated post-mortems ("fixed 2026-05-30…"), session logs, "Session N
DONE" status, or narration of what changed. **That is what git history is for.**
Once a fix lands and the code enforces it, delete the war story — keep only the
one-line rule that stops the next person undoing it.

**CLAUDE.md vs. memory vs. git:**
- **CLAUDE.md / AGENTS.md** — repo invariants and constraints (this file).
- **Agent memory** — facts about *Matt* and how he wants work done, and project
  context not derivable from the repo. Never duplicate repo facts into memory.
- **git history / commit messages** — the record of what changed and when.

When in doubt, prefer fewer words. A 300-line doc that is true beats a 1300-line
doc that is half stale.

---

## What this project is

A working PS3 port of [ioquake3](https://github.com/ioquake/ioq3) on the PSL1GHT
homebrew SDK. PS3 has no GPU OpenGL driver, so the project ships a custom
GL-1.1 → RSX/GCM translation layer (`code/gl/`). All upstream ioq3 sources are
**vendored under `code/` and patched in place** — there is no external `../ioq3`
checkout and no patch-script step. One source tree builds **five** PKGs (see
below). It boots, renders, plays online/LAN/bots, has DS3 input + rumble, OSK,
USB keyboard/mouse, cinematics, and OGG music.

---

## Build

- Requires `PS3DEV` set and the **devkitPro MSYS2** shell (`ppu-gcc`, GCC 7.2,
  LP64). You probably cannot build on the dev host — reason from source when the
  toolchain is unavailable. Matt builds and tests on hardware himself.
- Run all `make` from MSYS2 bash with `PS3DEV` set.

| Goal | Variant | `-D` flags | Build dir | TITLE_ID |
|---|---|---|---|---|
| `make pkg` | ioQuake3 (Q3A) | *(none)* | `build/` | `IOQ3PS300` |
| `make oa` / `make OA=1 pkg` | Open Arena | `STANDALONEOA` | `build_oa/` | `IOOAPS300` |
| `make ta` / `make TA=1 pkg` | Team Arena | `STANDALONETA` | `build_ta/` | `IOTAPS300` |
| `make classic` / `make CLASSIC=1 pkg` | Quake 3 Classic | `CLASSIC LEGACY_PROTOCOL` | `build_qc/` | `IOQCPS301` |
| `make ef` / `make EF=1 pkg` | Elite Force | `ELITEFORCE LEGACY_PROTOCOL` | `build_ef/` | `IOEFPS300` |

- All TITLE_IDs are **9 characters** — `CONTENT_ID`/PARAM.SFO APP_ID parsing on
  PS3 firmware expects exactly that length; a shorter/longer ID breaks PKG install.
- `make all-flavors` builds all five PKGs (Q3/TA/OA/Classic/EF) in sequence.
- `make clean` wipes all build dirs. Append `pkg` (installable PKG), `self`
  (raw SELF), or `install` (FTP-ready dir).
- `make DEBUG=1` adds `-DPS3_DEBUG -g`, enables the `ps3_log()` file and
  `com_logfile 2`. Release writes no log file.
- Icons come from `icons/<q3|oa|ta|qc|ef>/ICON0.PNG`.

---

## Editing rules

- Edit `code/<subdir>/<file>` directly — vendored sources are the source of truth.
- `ps3_*` files and everything in `code/gl/ audio/ input/ renderer/ spu/` are
  PS3-original — edit freely.
- Files with upstream ioq3 names are vendored — keep changes minimal, guard with
  `#ifdef __PS3__` where practical.
- All CLASSIC-only changes must be inside `#ifdef CLASSIC` and must not affect
  Q3/OA/TA builds.
- Stay at **`-O2`** globally — `-O3` breaks boot on this toolchain.
- Keep **`-mno-altivec`** globally — AltiVec is enabled per-file only for
  `ps3_snd.c`.
- All `.sh` scripts must be **LF** — CRLF breaks bash on the toolchain.
- Don't add a CD-Key entry UI — there's no input device flow for it, and
  `cl_cdkey` is pre-seeded.
- No hack-ish workarounds. This is a console port with strict platform rules;
  anything the platform doesn't expect crashes or regresses. When stuck, read the
  sibling PS4 port (see References) before guessing.

---

## Memory budget — hard constraint

- **~145 MB free user RAM** at boot (GameOS reserves ~88 MB of the 256 MB XDR).
  Logged at boot as `[mem] free user memory at Sys_PlatformInit: ~N MB`.
- Hunk = **96 MB**, Zone = **24 MB** → 120 MB, ~25 MB margin for QVM/sound/caches.
- **Do not raise hunk above 112 MB** without re-measuring. **zone = 32 MB hangs at
  boot** — keep `DEF_COMZONEMEGS = 24`.
- Constants live in [code/sys/ps3_platform.h](code/sys/ps3_platform.h) as
  **integers, not strings** (`common.c` does arithmetic on them).
- `common.c`'s `DEF_COMHUNKMEGS` / `DEF_COMZONEMEGS` are `#ifndef`-guarded so the
  PS3 overrides win — do not remove those guards.
- `MAX_CLIENTS` is effectively **64** regardless of `-DMAX_CLIENTS=8`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mayo1970/IoQuake3-PS3](https://github.com/Mayo1970/IoQuake3-PS3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

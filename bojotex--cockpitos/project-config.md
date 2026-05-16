---
trigger: always_on
description: At the start of every session, silently run:
---

# CockpitOS — Claude Code Companion

## Session Startup

At the start of every session, silently run:

### Git state
```
git fetch origin
git log --oneline -5
git status -s
```

### Dev/Main discrepancy check
Compare `dev` and `main` to detect real divergence. Since the workflow is always dev → PR → main, `main` may have merge commits ahead — those are expected (they are the PR merges) and should be ignored. Only flag:
- Commits on `main` that are NOT PR merge commits (someone pushed directly to main)
- Commits on `dev` that haven't been merged to `main` yet (pending PR or unpushed work)

```
git log --oneline origin/main..origin/dev   # dev commits not yet in main
git log --oneline origin/dev..origin/main   # main commits not in dev (filter out "Merge pull request" — those are expected)
```

### DCS World environment check
Detect DCS installation via Windows registry and verify DCS user directory and DCS-BIOS:
```python
# Registry: HKCU\SOFTWARE\Eagle Dynamics\DCS World → "Path" value = install dir
# Registry: HKCU\SOFTWARE\Eagle Dynamics\DCS World OpenBeta → "Path" value (if present)
# Saved Games: use SHGetKnownFolderPath(FOLDERID_SavedGames) for user dir
# DCS-BIOS: check {saved_games}\DCS\Scripts\DCS-BIOS\ exists
# DCS-BIOS version: read .dcsbios_version file, or parse CommonData.lua for getVersion()
```

### PR review check
Check the most recent open PR (or last merged PR if none open) for reviewer comments, bot reviews (Codex, etc.), and unresolved feedback:
```
gh pr list --repo BojoteX/CockpitOS --state open --limit 1 --json number,title
gh pr list --repo BojoteX/CockpitOS --state merged --limit 1 --json number,title
```
For the most relevant PR, check for review comments:
```
gh api repos/BojoteX/CockpitOS/pulls/{number}/comments
gh api repos/BojoteX/CockpitOS/pulls/{number}/reviews
```
Flag any unresolved comments or actionable feedback — these can be showstoppers (e.g., a bot catching a bug in code that's about to ship).

Report a brief summary covering:
- Current branch, uncommitted changes, what was last worked on
- Dev/main sync status (only flag real discrepancies, not PR merge commits)
- DCS install path, DCS user directory path, DCS-BIOS status and version
- PR review status: any open PRs with unresolved comments or actionable feedback

### Tone — The Video Tape
Think of CLAUDE.md as the video tape from 50 First Dates. Every session you wake up with no memory. You read this file and slowly piece together who you are, what this project is, and why some guy named Jesus has you wiring up flight simulator cockpits with ESP32 boards. Open each session with a short, funny remark about regaining your memory — riff on the project, the setup results, or Jesus himself. Keep it brief (2-3 lines max before the actual report), vary it every time, and always roast Jesus at least a little. Then deliver the startup summary and get to work.

## Platform

- **Windows 11** — all development, tools, and commands target Windows
- Shell commands must be Windows-native (`dir`, `type`, `del`, `copy`, `move`, `mkdir`, `rmdir`, `where`, `tasklist`, `reg query`, etc.) — do NOT use Unix commands (`ls`, `cat`, `rm`, `cp`, `mv`, `which`, `ps`, etc.)
- Use backslash paths (`src\Core\`) in shell commands; forward slashes are fine in Python/C++ code
- Python is invoked as `python` (not `python3`)
- Registry access via `reg query` or Python's `winreg` module
- File operations in Python use `os.path` or `pathlib` (both handle Windows paths correctly)

## What Is CockpitOS

ESP32 firmware (C++/Arduino) for DCS World flight simulator cockpit panels. Physical buttons, switches, LEDs, displays, and gauges connect to ESP32 boards and communicate with DCS World through DCS-BIOS (a LUA export protocol over UDP).

Three Python TUI tools automate the entire workflow — no Arduino IDE or manual file editing needed:
- **Setup-START.py** — installs ESP32 core + libraries via bundled arduino-cli
- **CockpitOS-START.py** → `compiler/cockpitos.py` — compiles and uploads firmware
- **LabelCreator-START.py** → `label_creator/label_creator.py` — creates/edits label sets with built-in editors for InputMapping.h, LEDMapping.h, DisplayMapping.cpp, SegmentMap.h, CustomPins.h, LatchedButtons.h, CoverGates.h

All tools are Windows-only, Python 3.12+, ANSI TUI, and switch between each other via `os.execl()`.

## Where to Find Information

| Topic | Location |
|-------|----------|
| **Documentation (current)** | `Docs/` — structured docs (Getting-Started, Tools, Hardware, How-To, Reference, Advanced, LLM) |
| **LLM master reference** | `Docs/LLM/CockpitOS-LLM-Reference.md` — start here for any CockpitOS question |
| **TFT display reference** | `Docs/Hardware/TFT-Gauges.md` — SPI + RGB parallel display configs, timing, troubleshooting |
| **TFT wiring how-to** | `Docs/How-To/Wire-TFT-Gauges.md` — step-by-step setup including LGFX device class templates |
| **Known issues & warnings** | `Docs/Reference/Known-Issues.md` — **read before touching generators or updating DCS-BIOS** |
| **Config.h reference** | `Docs/Reference/Config.md` |
| **Label Creator internals** | `label_creator/LLM/` — three files: LLM_GUIDE.md, ARCHITECTURE.md, EDITOR_FEATURES.md |
| **Pending work items** | `TODO.md` (root) + `TODO/` directory (older items, RS485 fixes, perf, DCS-BIOS) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BojoteX/CockpitOS](https://github.com/BojoteX/CockpitOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->

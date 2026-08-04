---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A desk device that shows live Claude Code session status via a blocky orange Claude mascot.
Hardware: ESP32-2432S028R ("Cheap Yellow Display"), ILI9341 320x240 landscape, BLE only.

`DESIGN.md` (Thai) records the **current** design and the reasons that are not visible from
the code — not a changelog. **Read the relevant section before changing visuals, protocol,
or layout.** Write to it only when a previous decision is *reversed*, or when you hit a
constraint the code cannot state on its own (a measured hardware value, a framework quirk, a
rule that must hold in two places at once). Ordinary fixes need no entry — git history is
the log. When an entry stops being true, rewrite it in place; do not append the correction
below it.

## Data flow

```
Claude Code hooks --> tamaclaude --hook --> Unix socket --> daemon --> BLE GATT --> board

Claude Code statusline --> ~/.tamaclaude/statusline.sh --.
                                                          >--> ~/.claude/.statusline-usage-cache
menu bar timer --> tamaclaude --usage-poll --> claude.ai --'                |
                                                        daemon reads --> "u" key --> board
```

The quota panel has **two** sources, not one. The statusline pipe needs no credential but is
event-driven, so it goes quiet exactly when the desk display is left alone; the poll pipe uses
the user's `sessionKey` and keeps the number moving with Claude Code closed. Neither replaces
the other and they are separate switches — see the reversal note in `DESIGN.md`.

The daemon owns all logic. Firmware only knows a fixed `VisualState` enum and draws it.
Tool-to-animation mapping is host-side and user-overridable at `~/.tamaclaude/tools.json`.

## Commands

### Host (Swift, macOS 14+)

```bash
cd host
swift build                        # debug
swift run tamatest                 # run the whole test suite
swift run tamaclaude --daemon --print --no-ble -v   # daemon without bluetooth, prints snapshots
swift run tamaclaude --send '<json>'                # inject one hand-written hook event
swift run tamaclaude --usage-poll                   # one quota fetch -> cache, then exit
swift run tamaclaude --usage-cache < statusline.json  # the statusline pipe, by hand
swift run tamaclaude --install-statusline           # take over statusLine.command
swift run tamaclaude --remove-statusline            # give the slot back
./Scripts/make-app.sh              # release .app -> host/dist/TamaClaude.app
./Scripts/make-app.sh --install    # install to /Applications and launch
```

`--usage-poll` reads the key from `~/.tamaclaude/session-key` (mode 600, never argv, never env)
and exits: `0` = wrote the cache · `2` = key rejected · `3` = key file unusable · `1` = anything else.
The menu bar app runs it on a timer; the key is set from its gear menu, not by hand.

There is **no `testTarget`** and no per-test filter — `swift run tamatest` runs everything
(`Sources/tamatest/Tests.swift`, grouped by `suite("...")`). A machine with only Command Line
Tools would build a `testTarget` and exit 0 without running it, which is worse than no tests.
To narrow the run, temporarily comment out `suite(...)` calls in `runAllTests()`.

Bluetooth only works from the `.app` launched via LaunchServices (`open`) — a bare binary run
from a shell gets `SIGABRT` from TCC, not a polite denial. Every rebuild changes the adhoc
cdhash, so macOS re-asks for Bluetooth permission.

### Firmware (ESP-IDF v5.5)

```bash
cd firmware
idf.py -p /dev/cu.usbserial-XX flash monitor
```

`firmware/probe/` is a separate throwaway IDF project that interrogates the real panel
(MADCTL, colour order, inversion). Its findings are recorded in `DESIGN.md`; the firmware
uses those constants, not a chip model number.

### Graphics / preview (Python + Pillow)

```bash
python3 tools/preview.py            # render every state + whole screens to out/ (PNG + GIF)
python3 tools/preview.py --sheet    # contact sheet only
python3 tools/export_layout.py      # tools/layout.toml -> firmware/main/layout.h
python3 tools/make_icon.py          # logo PNG (≥128px) + mascot (≤64px) -> host/Resources/AppIcon.icns
```

There is no SDL2 simulator. `tools/preview.py` is the visual dev loop: change a rect, render,
look at `out/`. It proves the *design*, not the C renderer.

## Architecture

### One source of truth per concern

- **`tools/layout.toml`** — every layout constant, palette colour, and randomised table
  (star/grass/cloud positions). Python reads it via `tools/gen/config.py`; C gets it through
  the generated `firmware/main/layout.h`. **Never edit `layout.h`** — edit the TOML and rerun
  `export_layout.py`. If preview and board disagree, that is a renderer bug, by construction.
- **`tools/gen/*.py` ↔ `firmware/main/ct_*.c`** — deliberate parallel ports, file for file:
  `props.py`↔`ct_props.c`, `mascot.py`↔`ct_mascot.c`, `rects.py`↔`ct_rects.c`,
  `screen.py`↔`ct_ui.c`, `sky.py` folds into `ct_ui.c`. A visual change means editing both
  sides; the Python side is where you iterate, the C side is the port.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thaitop/tamaclaude](https://github.com/thaitop/tamaclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->

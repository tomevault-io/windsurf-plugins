---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this project.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this project.

## Project Overview

**Wind Waker Multiplayer** — a Go-based tool that enables real-time visual
multiplayer in The Legend of Zelda: The Wind Waker on Dolphin emulator.
Each player sees the other's actual Link in-game, walking around at the
remote's real world coords with their real animations, ~50ms latency on
LAN. Two-Dolphin local play is wired up via `scripts/mplay2.sh`.

The mod is shipped as a standalone `ww-multiplayer.exe` patcher (`./ww-multiplayer.exe patch
<vanilla.iso>` produces the patched ISO from the user's own legitimate
Wind Waker disc image). Releases are cut on tag push via GitHub Actions.

## Repository Structure

```
ww-multiplayer/
├── main.go                      # Entry point: TUI + all CLI subcommands
├── internal/
│   ├── dolphin/                 # Dolphin process memory access (Win32)
│   │   ├── memory.go            # Core read/write, RAM scanner
│   │   ├── inject.go            # LEGACY runtime injection (vestigial; superseded by inject/ patched-ISO approach)
│   │   ├── inject_code.go       # LEGACY PPC blob (vestigial)
│   │   └── helpers.go
│   ├── inject/                  # Standalone ISO patcher (used by `./ww-multiplayer.exe patch`)
│   │   ├── blob.go              # AUTO-GENERATED via scripts/extract_blob.py
│   │   ├── dol.go               # DOL header editor + T2 splice + in-DOL patches
│   │   ├── iso.go               # ISO patcher with FST relocation
│   │   └── ciso.go              # CISO decompressor
│   ├── network/                 # TCP server/client + protocol
│   │   ├── protocol.go
│   │   ├── server.go
│   │   └── client.go
│   ├── report/                  # Reporter interface (Stdout / Discard / TUI impls)
│   │   └── report.go
│   └── tui/                     # Charm Bubble Tea UI (resurrected v0.1.5)
│       ├── app.go, splash.go, connect.go, dashboard.go, session.go, styles.go
├── inject/                      # C source for the injected PPC code
│   ├── src/multiplayer.c        # The mod's C side
│   ├── include/{game,mailbox}.h
│   ├── build.py                 # Freighter wrapper — builds patched.dol from original.dol
│   └── patch_iso.py             # Local-dev ISO splicer (relies on `wit copy`-prepped ISO)
├── cheats/
│   └── GZLE01.ini               # Curated Gecko code pack, copied into <USER_DIR>/GameSettings by `dolphin2`
├── scripts/
│   └── extract_blob.py          # Diffs original.dol vs patched.dol → internal/inject/blob.go
├── .github/workflows/
│   ├── build.yml                # CI: go vet + cross-build on push/PR
│   └── release.yml              # CI: build + release on tag push
└── docs/                        # IMPORTANT - read before debugging
    ├── 01-architecture.md, 02-dolphin-memory.md, 03-code-injection.md
    ├── 04-ww-addresses.md, 05-known-issues.md, 06-history.md
```

## Commands

```bash
# Build
go build -o ww-multiplayer.exe .

# End-user entry points
./ww-multiplayer.exe                                    # Launch TUI (host or join)
./ww-multiplayer.exe patch <iso|ciso> [out.iso]         # Splice mod into user's own vanilla WW ISO

# Multiplayer runtime CLIs (used by scripts/mplay2.sh)
./ww-multiplayer.exe server                             # Headless TCP server on :25565
./ww-multiplayer.exe broadcast-pose <name> <addr>       # Stream this Dolphin's Link pose+pos to server
./ww-multiplayer.exe puppet-sync <name> <addr>          # Receive remotes; render them as Link #2 / actor puppets
./ww-multiplayer.exe broadcast-link <name> <addr>       # Position-only broadcast (cheaper; no pose)
./ww-multiplayer.exe pose-fake-loop <name> <addr>       # Loopback dev: capture pose once, stream as a fake remote
./ww-multiplayer.exe pose-test [mirror|freeze] [secs]   # Single-Dolphin sanity test for the pose pipeline

# Diagnostics
./ww-multiplayer.exe screenshot [path]                  # PNG of the selected Dolphin's window (Win32; default path = dolphin-<pid>-<ts>.png)
./ww-multiplayer.exe input <btns-hex> <stickX> <stickY> [ms=1000]   # Drive synthetic controller input via pad_read_shim. ms=0 holds until input-release.
./ww-multiplayer.exe input-release                      # Disable pad_read_shim override (zero input_enable). Pair with `input ... 0`.
./ww-multiplayer.exe auto-recapture [out=saves/start.sav]   # Cold-boot Dolphin + drive menus + prompt for one Shift+F1 + cp the new state. Win-only.
./ww-multiplayer.exe auto-recapture-pair [out1=saves/start.sav] [out2=saves/start2.sav] [delay-secs=5]   # Same as auto-recapture, but captures TWO states with a delay between (distinct btp phases). Pair with SAVE_STATE_2 in dolphin2 to validate face-sync (#5).
./ww-multiplayer.exe send-shift-f1                      # Diagnostic: probe whether your Dolphin build accepts synthetic Shift+F1 hotkeys
./ww-multiplayer.exe debug                              # Print Link's position for 5 sec
./ww-multiplayer.exe dump                               # Dump mailbox state (shadow_mode, pose seqs, etc.)
./ww-multiplayer.exe check                              # Mailbox + player pointers + BSS sanity check

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StephenSHorton/ww-multiplayer](https://github.com/StephenSHorton/ww-multiplayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

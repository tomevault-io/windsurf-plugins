---
trigger: always_on
description: Guidance for AI coding agents working on **DemoTape** — build, run, verify, troubleshoot.
---

# AGENTS.md

Guidance for AI coding agents working on **DemoTape** — build, run, verify, troubleshoot.
(For contribution process, PR standards, and review expectations, see `CONTRIBUTING.md`.)

## What this is

A native macOS menu-bar screen recorder (Swift + AppKit + AVFoundation + Core Image/Metal)
that auto-styles recordings (auto-zoom, smooth cursor, webcam, framed backgrounds) and
exports lightweight web MP4s. **No external dependencies** — Apple frameworks only. **No
Xcode project** — it's a Swift Package built from the command line.

## Prerequisites

- macOS **12.3+** (Intel or Apple Silicon)
- Xcode **Command Line Tools**: `xcode-select --install`
- Optional: `gh` for release tasks

## Build, run, verify

```bash
# Compile (fast feedback; use this to check your changes build)
swift build -c release

# Run the unit tests (pure logic: caption parsing, SRT/VTT formatting, endpoints)
swift test

# One-time: create the stable self-signed signing identity ("DemoTape Dev").
# Required so macOS remembers Screen Recording permission across rebuilds.
./create-identity.sh

# Package + sign + install to /Applications, then launch
./build-app.sh release
open /Applications/DemoTape.app
```

> Always run the app from **/Applications** (build-app.sh installs it there). macOS grants
> Screen Recording permission unreliably to apps in Desktop/Documents/Downloads.

## Agent-assisted setup for a tester (run this on the tester's Mac)

This is the recommended way to try DemoTape without a notarized download. Building on the
tester's own machine produces a **native-arch** binary (no Rosetta), signs it with a **stable
local identity** (so Screen Recording permission survives updates), and involves **no
Gatekeeper/quarantine prompt** because nothing is downloaded and run — it's built locally.

**If you are the human tester:** paste the prompt below to your coding agent (Claude Code,
Codex, etc.) from inside a clone of this repo, and let it run the steps.

> Set up and install DemoTape on my Mac by following the "Agent-assisted setup for a tester"
> runbook in AGENTS.md. Run the commands, check each precondition, tell me exactly when you
> need me to click something in System Settings, and stop with a clear message if any step
> fails.

**If you are the agent, run these steps in order and verify each one:**

```bash
# 0. Preconditions ----------------------------------------------------------
sw_vers -productVersion            # must be >= 12.3; abort with a message if lower
uname -m                           # arm64 (Apple Silicon) or x86_64 (Intel) — just informational
xcode-select -p >/dev/null 2>&1 || xcode-select --install   # installs Command Line Tools if missing
#   If it launched the CLT installer, STOP and tell the user to finish the GUI
#   installer, then re-run. Do not proceed until `xcode-select -p` succeeds.

#   IMPORTANT: `xcode-select -p` only proves the *path* is set — it does NOT prove
#   the toolchain is healthy. A partially corrupted Command Line Tools install
#   (common after a macOS upgrade) can pass every check above and still fail the
#   build with the cryptic error `no such module 'PackageDescription'` /
#   `Invalid manifest`. `swift --version` and even `swift package --version` still
#   succeed in that state, so they are NOT reliable checks. The only reliable test
#   is actually compiling a manifest — do that in a throwaway package:
_dt_probe="$(mktemp -d)"; printf '// swift-tools-version:5.7\nimport PackageDescription\nlet package = Package(name: "probe")\n' > "${_dt_probe}/Package.swift"
if ! swift package --package-path "${_dt_probe}" dump-package >/dev/null 2>&1; then
    rm -rf "${_dt_probe}"
    echo "SwiftPM cannot compile a manifest — your Command Line Tools install is corrupted"
    echo "(the PackageDescription module is missing). Fix it, then re-run this runbook:"
    echo "  sudo rm -rf /Library/Developer/CommandLineTools && xcode-select --install"
    echo "If that does not resolve it, install full Xcode and select it:"
    echo "  sudo xcode-select -s /Applications/Xcode.app"
    exit 1
fi
rm -rf "${_dt_probe}"
#   See the 'no such module PackageDescription' entry under Troubleshooting below.

# 1. Build + verify ---------------------------------------------------------
swift build -c release             # must succeed
swift test                         # 386 tests; all must pass (pure logic, no GUI/network)

# 2. Stable signing identity (one-time; persists Screen Recording permission) --
security find-identity -v -p codesigning | grep -q "DemoTape Dev" || ./create-identity.sh

# 3. Package + sign + install to /Applications ------------------------------
./build-app.sh release             # assembles .app, signs with "DemoTape Dev", installs
open /Applications/DemoTape.app    # a record icon appears in the menu bar
```

**Then hand off to the human for the one-time macOS permission grant (the agent cannot click
these):**

1. Click the DemoTape menu-bar icon and press **Start** once. macOS shows a Screen Recording
   prompt.
2. Open **System Settings → Privacy & Security → Screen Recording**, enable **DemoTape**.
3. **Quit and reopen** DemoTape (required — macOS only applies the grant on relaunch).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabosarmiento/demotape](https://github.com/gabosarmiento/demotape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->

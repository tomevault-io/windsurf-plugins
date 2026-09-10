---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

DelugeDisplay is a macOS SwiftUI app that mirrors the display of a Synthstrom Deluge groovebox over MIDI SysEx — either the 128×48 OLED or the 4-digit 7-segment display. Pure Apple frameworks (SwiftUI, AppKit, CoreMIDI, OSLog); no package manager and no third-party dependencies. Single app target `DelugeDisplay`, bundle id `net.dcarmich.DelugeDisplay`, deployment target macOS 15.4. Sandboxed with `device.usb` and `files.user-selected.read-write` entitlements (MIDI access + the screenshot save panel).

## Build & run

```sh
open DelugeDisplay.xcodeproj      # develop in Xcode; run with Cmd+R
xcodebuild -project DelugeDisplay.xcodeproj -scheme DelugeDisplay -configuration Debug build
```

- **There is no test target, linter, or formatter** — don't look for or invent test commands. Verification is manual: build and run against a Deluge connected via MIDI (without hardware you can still confirm the app builds and shows the "WAITING FOR DELUGE" banner).
- App version is `MARKETING_VERSION` in `project.pbxproj` (update both Debug and Release blocks).
- Releases are hand-distributed as .dmg/.zip via GitHub Releases; there is no CI in this repo.

## Branches

- `main` — uses the firmware's **subscription/push** display mode: the app renews a 2 s lease and the Deluge pushes RLE deltas only when its screen changes, with periodic and on-error full-image resyncs on the app side. (Historically `main` was full-frame polling "for Ethernet compatibility"; the delta failures behind that rule were traced to since-fixed transport bugs in this app — `MIDIPacketList` value-copying, chunk reordering, interleaved-message corruption — not to the delta protocol. An old `ios-delta` experiment branch was superseded by this and deleted.)

## Architecture

Everything flows through **`MIDIManager`** (`MIDIManager.swift`), a `@MainActor ObservableObject` owning all non-view state: the CoreMIDI client/ports, port scanning and auto/re-connect, the SysEx protocol, the connection state machine, and the published display state (`frameBuffer`, `sevenSegmentDigits`/`sevenSegmentDots`, `isConnected`, plus user prefs — display mode, color mode, smoothing, pixel grid). `DelugeDisplayApp` creates it as a `@StateObject` and injects it with `.environmentObject`; views read it and set properties on it, never talk MIDI themselves.

### MIDIManager is a single, shared instance
`DelugeDisplayApp` owns the only `MIDIManager` (`@StateObject`) and hands it to `AppDelegate` via `.onAppear` (`appDelegate.midiManager`), whose `applicationWillTerminate` disconnect acts on it. Never construct a second manager — a second instance opens its own CoreMIDI client that connects and subscribes to the Deluge in parallel.

### SysEx protocol (manufacturer id 0x7D)
Requests (constants at the bottom of `MIDIManager`):
- `F0 7D 02 00 02 F7` — subscribe/renew the display push lease (2 s on the firmware side); `F0 7D 02 00 03 F7` — same, but forces a full-image resend. Steady state is renewals only; forces are deliberately rare (see lifecycle §3).
- `F0 7D 02 00 01 F7` — request one full OLED frame (connection probe / `connectionTimer`)
- `F0 7D 02 01 00 F7` — request 7-segment data (probe)
- `F0 7D 02 00 04 F7` — toggle which display the Deluge hardware shows (sent on every mode switch)

Responses (parsed in `processSysExMessage`; bytes are reassembled between `F0`/`F7` in `reassembleSysEx` on `processQueue`, which skips interleaved realtime bytes and whole channel/system-common messages):
- OLED full image: `F0 7D 02 40 01 00 …` — payload from byte 6 is RLE-packed; `unpack7to8RLE` (`RLEDecoder.swift`) must expand it to exactly 768 bytes or a resync is scheduled.
- OLED delta (push): `F0 7D 02 40 02 <start> <len> …` — start/len in 8-byte units into the 768-byte buffer; payload from byte 7 must unpack to exactly `8*len` bytes or a resync is scheduled.
- 7-seg: exactly 12 bytes, `F0 7D 02 41 …` — byte 6 is the dot bitmask, bytes 7–10 the segment bitmasks for the 4 digits (bit 6 → segment A … bit 0 → segment G, decoded in `SevenSegmentDigitView`).

Frame buffer layout (SSD1306-style pages): 768 bytes = 6 blocks × 128 columns; each byte is a vertical 8-pixel strip, `byteIndex = block * 128 + col`, bit n = row within the block.

### Stress test panel (display SysEx family 0x03)
Host side of the firmware's display link stress exerciser (spec: `docs/deluge-firmware-spec-display-midi-exerciser.md`; firmware branch `feature/display-stress-exerciser`). MIDI menu → "Stress Test…" (⇧⌘T) opens a `Window` scene backed by `StressTester` (model, owned lazily by `MIDIManager`) + `StressTestView`.
- Requests `F0 7D 02 03 <sub> … F7`: `00` stop, `01` start (`shape, interval, p1, p2, seed[4], capSecs[2]`, all LSB-first 7-bit), `02` query stats, `03` reset stats. Replies `F0 7D 02 43 <sub> …`: acks carry a status byte (0 ok / 1 bad param / 2 refused / 3 no OLED); stats carry a layout-version byte then a `pack_8bit_to_7bit` payload (plain Sequential packing — decoded by `StressTester.unpack7to8`, distinct from the RLE codec).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [douglas-carmichael/DelugeDisplay](https://github.com/douglas-carmichael/DelugeDisplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

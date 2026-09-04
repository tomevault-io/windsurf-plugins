---
trigger: always_on
description: Guidance for Claude Code (and future contributors) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (and future contributors) working in this repository.

## What this project is

InterDesk (renamed 2026-09 from BKMD, "Bluetooth Keyboard Mouse Dongle" — the old name still
survives in internal identifiers, see "Conventions & gotchas") lets you control one computer
("**PC2**", the target) using the keyboard (and eventually mouse) of another computer ("**PC1**",
typically a laptop), over BLE, via a custom ESP32-S3 USB dongle.

Motivating use case: a desktop PC + a laptop set up side by side, where the laptop is physically
in front of/on top of the desktop's keyboard, leaving no room to use it. Instead of reaching
around, the user types on the laptop and those keystrokes are forwarded wirelessly to the desktop.

### Physical/data flow

```
Laptop (PC1)                         Desktop/target PC (PC2)
┌─────────────────────┐              ┌──────────────────────┐
│ Electron app         │   BLE        │  USB port              │
│  - KeyMonitor         │───────────▶│  ┌──────────────────┐  │
│    (uiohook-napi)     │  writes    │  │ ESP32-S3 dongle    │  │
│  - BluetoothManager   │  HID       │  │  - NimBLE server   │  │
│    (@stoprocent/noble)│  reports   │  │  - USB HID device  │──┼─▶ appears as a
└─────────────────────┘              │  │    (keyboard/mouse)│  │   real keyboard/mouse
                                      │  └──────────────────┘  │   to PC2
                                      └──────────────────────┘
```

1. The Electron app runs on the laptop (PC1) and hooks global keyboard events with
   `uiohook-napi`.
2. Keys are translated into standard USB HID usage codes and packed into an 8-byte HID
   boot-keyboard report.
3. The report is written over BLE to a characteristic exposed by the ESP32 dongle, which is
   plugged in via USB-A to PC2.
4. The dongle firmware (NimBLE peripheral + USB HID device) receives the report and replays it
   over USB HID, so PC2's OS sees a normal hardware keyboard/mouse.
5. A global shortcut in the Electron app (`Cmd/Ctrl+Shift+R`) toggles whether local keystrokes
   are currently being captured/forwarded — this is the "switching" mechanism referenced
   throughout the code, meant to avoid sending input to both machines at once.

## Repo layout

```
app/            Active cross-platform desktop app (Electron + React + Tailwind + TypeScript)
firmware/
  InterDesk_firmware/    Active ESP32-S3 firmware (PlatformIO + Arduino framework + NimBLE)
  platformio/            Local scratch PlatformIO scaffold, gitignored, not part of the build
docs/           Project docs; docs/reference/deskhop/ holds the DeskHop analysis (see below).
                Its reference/, sessions/ and getting-started.md are gitignored — see the note
                below.
img/            README assets
archive/        Retired, not maintained — ignore unless asked. Gitignored. macOS-prototype/
                (the frozen Swift/SwiftUI app) and python-scripts/ (early spike scripts).
```

**Local-only paths (gitignored as of 2026-09).** `archive/`, `docs/reference/`, `docs/sessions/`
and `docs/getting-started.md` were untracked and added to `.gitignore` — they live on the primary
maintainer's machine only. A fresh clone will not have them, so every reference to them in this
file (including the DeskHop section below) is a dead pointer outside that working copy. If you are
on a clone that lacks them, say so instead of guessing at their contents; ask the maintainer to
share the file.

Ownership split: the desktop app is developed by the primary maintainer
(lexatuan@gmail.com); firmware is developed by hardware collaborator **@Dubleriino**. Firmware
source comments are frequently written in Czech.

## Current implementation status (as of 2026-08)

Working:
- Electron app: device discovery and connection UI, global shortcut start/stop of keyboard and
  mouse capture, and forwarding HID reports over BLE to a dongle.
- DeskHop-style **dynamic switching** (implemented 2026-08; hardware-verified 2026-08-30 — PC2
  accepts the absolute HID descriptor and the cursor tracks PC1 motion 1:1): a virtual
  cursor in 0..32767 space (`mousemonitor.ts` absolute mode), edge-crossing detection on PC1
  (`edge-switcher.ts`), and an absolute-pointer USB HID device on the dongle
  (`firmware/InterDesk_firmware/src/usb/abs_mouse.*`). Settings: `dynamicSwitch`, `pc2Side`,
  `mouseMode` ('absolute'|'relative'). The global shortcut remains as manual switching.
- Firmware: receiving 8-byte keyboard, 4-byte relative-mouse, and 6-byte absolute-mouse reports
  over BLE and replaying them as USB HID; "AirDrop" advertising toggle via long button press;
  optional TFT status display on the LilyGO board variant.

Not yet working / explicitly TODO in code:
- **Local cursor suppression while forwarding**: uiohook only observes the cursor, so while
  forwarding the local macOS cursor pins at the physical screen edge and coordinate-derived
  deltas collapse to zero there (limits movement away from the entry edge on PC2). Planned fix:
  a pointer-capture overlay window feeding `MouseMonitor.applyDelta()` (already isolated for it).
- Screen-position calibration UI exists (`src/ui/components/SwitchingPage.tsx` — drag-to-arrange

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lexa105/InterDesk](https://github.com/lexa105/InterDesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->

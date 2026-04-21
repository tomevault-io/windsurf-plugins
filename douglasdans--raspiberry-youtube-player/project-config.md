---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YouTube TV kiosk for Raspberry Pi OS Bookworm (64-bit Lite). Opens `youtube.com/tv` in Chromium (kiosk mode via Cage/Wayland) and translates gamepad inputs into keyboard events so YouTube TV's keyboard-based navigation works with physical controllers.

**Target hardware:** Raspberry Pi 4 or 5 running Raspberry Pi OS Bookworm Lite (no desktop environment).

## Why only YouTube (and not Netflix/Disney+)

YouTube exposes `youtube.com/tv` — a public URL maintained by Google that delivers the Leanback interface, the same one used on Google TV and Chromecast. Accessing it with the right user-agent (Samsung Tizen) forces the TV interface server-side.

Netflix, Disney+ and HBO Max have **no equivalent**. Their TV interface is a native app (Android TV, Fire TV) and has never been exposed as a public webapp. Running Waydroid on the Pi for emulated Android is theoretically possible but Widevine certification on non-certified devices blocks Netflix specifically. There is no clean solution for these services on Pi OS Lite today.

## Architecture

```
Physical gamepad
  → pygame/SDL2 (gamecontrollerdb.txt identifies the controller)
    → gamepad_mapper.py (translates SDL2 button/axis events → keyboard keycodes)
      → uinput (injects synthetic key events into the kernel)
        → Cage (Wayland compositor routes events to Chromium in focus)
          → youtube.com/tv
```

**Key design decision:** `youtube.com/tv` has no Gamepad API support — it navigates purely via keyboard (like Android TV / PlayStation). The daemon intercepts the gamepad at the OS level via SDL2 and re-emits keyboard events via uinput, making the translation invisible to Chromium.

**Why `SDL_VIDEODRIVER=dummy`:** The mapper runs headless (no display), so SDL2 must not try to open a window. Both `SDL_VIDEODRIVER` and `SDL_AUDIODRIVER` must be set *before* `import pygame`.

**Why Cage:** Minimal Wayland compositor that accepts a single fullscreen process. Works on Pi OS Lite without a desktop environment.

**User-agent trick:** Chromium spoofs a Samsung Smart TV user-agent so `youtube.com/tv` recognises it as a supported TV browser and delivers the Leanback interface instead of redirecting to youtube.com.

**Current user-agent:** `Tizen 6.0 / SamsungBrowser/3.1` — researched in March 2025 as potentially outdated. Tizen 9.0 with Chrome/120 is more reliable for the YouTube TV interface to be consistently delivered. Updating this is a pending improvement.

## Button mapping

| SDL2 Button         | Emitted key    | YouTube TV action            |
|---------------------|----------------|------------------------------|
| D-pad ↑↓←→          | Arrow keys     | Navigate / seek ±5s          |
| Left stick          | Arrow keys     | Same as D-pad (with threshold)|
| A / Cross           | Enter          | Select                       |
| B / Circle          | Backspace      | Back (some contexts use Escape)|
| X / Square          | Space          | Play / Pause                 |
| Y / Triangle        | F              | Fullscreen                   |
| LB / L1             | J              | Seek −10s                    |
| RB / R1             | L              | Seek +10s                    |
| L3                  | M              | Mute                         |
| R3                  | S              | Search                       |
| SELECT + START      | —              | Kill Chromium and exit daemon |

Direction keys have hold-repeat (delay 0.4s, rate 0.08s). Other keys do not.

## Running

Install (Raspberry Pi only, requires sudo):

```bash
bash install.sh
```

Start the full kiosk (after install):

```bash
bash ~/scripts/youtube.sh
```

Test gamepad detection standalone (no Chromium needed):

```bash
python3 ~/scripts/gamepad_mapper.py
# or, from repo root before installing:
python3 gamepad_mapper.py
```

## Key Configuration

Tuning variables at the top of `gamepad_mapper.py`:

| Variable         | Default | Effect                              |
|------------------|---------|-------------------------------------|
| `AXIS_THRESHOLD` | `0.5`   | Analog stick dead zone (0.0–1.0)    |
| `REPEAT_DELAY`   | `0.40`  | Seconds before hold-repeat starts   |
| `REPEAT_RATE`    | `0.08`  | Seconds between repeated key taps   |
| `BUTTON_TO_KEY`  | dict    | Full button → evdev keycode mapping |

SDL2 button constants (`BUTTON_A`, `BUTTON_B`, etc.) are defined explicitly in `gamepad_mapper.py` because `pygame._sdl2.controller` doesn't expose them as module attributes in all pygame versions. These values are fixed in the SDL2 spec and will never change.

## Dependencies

- `python3-pygame` — SDL2 gamepad input
- `python3-evdev` — uinput virtual keyboard output
- `chromium` — browser (package name on Bookworm; NOT `chromium-browser`)
- `cage` — minimal Wayland compositor (must be installed separately: `sudo apt install cage`; not in `install.sh`)
- udev rule: `/etc/udev/rules.d/99-uinput.rules` — allows `input` group to write `/dev/uinput`
- User must be in the `input` group (added by `install.sh`, requires re-login)

## Install Artifacts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DouglasDans) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

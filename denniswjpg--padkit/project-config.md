---
trigger: always_on
description: PadKit is an open-source **firmware + tooling package** for a small USB macropad:
---

# PadKit — the spec

PadKit is an open-source **firmware + tooling package** for a small USB macropad:
**6 mechanical keys + 1 rotary encoder with a push switch**, built around a
**WCH CH552G** (8051-core) microcontroller, with a strip of **6 addressable RGB
LEDs** (WS2812 / NeoPixel-style).

This file is the load-bearing document. It is written to be read equally by a
human and by a coding agent that a user points at their own setup. If you are an
AI agent asked to "wire the PadKit into my system," everything you need — the
exact keycodes, the LED report bytes, and worked binding examples — is here. Read
the [For AI agents](#for-ai-agents) section last; read the input map first.

The single most important design fact: **the pad is a plain USB keyboard that
types F13–F23.** Those function keys exist in every OS's key tables but no normal
keyboard ever sends them, so you can bind them to anything without stealing real
keystrokes — **with no driver and no software of any kind.**

---

## Contents

- [Usage tiers (what ships vs what's planned)](#usage-tiers)
- [The input map](#the-input-map) — every key/gesture → keycode
- [Binding the keys](#binding-the-keys) — macOS, Windows, Linux, Home Assistant, etc.
- [LED control protocol](#led-control-protocol) — HID reports to drive the RGB
- [Flashing & recovery](#flashing--recovery)
- [For AI agents](#for-ai-agents) — worked micro-examples
- [Roadmap / v0.2](#roadmap--v02)
- [Repository layout](#repository-layout)

---

## Usage tiers

PadKit is meant to be usable at three increasing levels of investment. **Only the
first tier ships in v0.1.** The other two are on the [roadmap](#roadmap--v02) and
are clearly labelled everywhere as *planned*, not shipped.

| Tier | What it is | Status |
|---|---|---|
| **1. Flash + bind** | Flash the firmware, then bind F13–F23 in your OS or app. No PadKit software runs on the host. | **Ships in v0.1** (firmware + `isp55e0` flasher + this spec). |
| **2. Browser config tool** | A zero-install web page (WebHID/WebUSB) to flash and to edit LEDs/keymap live. | **Roadmap (v0.2).** Feasibility confirmed; see roadmap. |
| **3. Companion daemon** | A small host service that turns pad events into actions and pushes LED state. | **Roadmap.** `daemon/` is a placeholder. |

Tier 1 is fully self-sufficient. You never need tiers 2–3 to use the pad — they
are convenience layers.

---

## The input map

The firmware presents a standard **boot-protocol USB keyboard** (no driver). Every
control maps to a function key in the **F13–F23** range.

| Control | Gesture | Key | HID usage | Event style |
|---|---|---|---|---|
| Key 1 | press / release | **F13** | `0x68` | real **key-down** on press, **key-up** on release |
| Key 2 | press / release | **F14** | `0x69` | real key-down / key-up |
| Key 3 | press / release | **F15** | `0x6A` | real key-down / key-up |
| Key 4 | press / release | **F16** | `0x6B` | real key-down / key-up |
| Key 5 | press / release | **F17** | `0x6C` | real key-down / key-up |
| Key 6 | press / release | **F18** | `0x6D` | real key-down / key-up |
| Knob | turn CW (one detent) | **F21** | `0x70` | tap (down+up), one per detent |
| Knob | turn CCW (one detent) | **F19** | `0x6E` | tap, one per detent |
| Knob | click (push) | **F20** | `0x6F` | tap, on release |
| Knob | push **and** turn CW | **F23** | `0x72` | tap, one per detent |
| Knob | push **and** turn CCW | **F22** | `0x71` | tap, one per detent |

All usages are on the **Keyboard/Keypad usage page (0x07)** — the standard USB HID
table, where F13=`0x68` runs consecutively to F23=`0x72`.

### Semantics that matter when you bind

- **Tap vs hold exists only for the six keys.** Keys 1–6 report a genuine
  key-down and a later key-up, so a host can time the interval and distinguish a
  *tap* from a *hold* (e.g. tap = action A, hold ≥ 400 ms = action B). This is a
  host-side decision — the firmware just reports the edges honestly.
- **Knob gestures are momentary taps.** Each turn detent, each click, and each
  push-turn detent is emitted as a quick down+up. There is no "held knob" key
  event; you get a discrete tap per detent. Bind them like button presses, not
  like keys you can hold.
- **Click is suppressed by a push-turn.** If you press the knob and turn it while
  pressed, you get F22/F23 for the turn and the **click (F20) is *not* emitted**
  on release. A clean press-and-release with no turn emits F20. This lets the knob
  carry two independent axes (turn = F19/F21, push-turn = F22/F23) plus a button
  (F20) without collisions.
- **CW/CCW labelling.** "CW" and "CCW" here follow the reference wiring
  (ENC_A=P3.1, ENC_B=P3.0). On a differently wired clone the two turn directions
  still map to F19 and F21 — they may just be swapped relative to physical
  rotation. If your knob feels "backwards," swap your F19/F21 bindings (or the
  encoder A/B pins). See [`docs/hardware.md`](docs/hardware.md).

---

## Binding the keys

Because the pad is an ordinary keyboard, you bind F13–F23 with whatever hotkey
mechanism you already use. **No PadKit software is involved in any of these.**
Below are concrete, correct examples per environment. Replace the actions with
your own.

### macOS — Karabiner-Elements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denniswjpg/padkit](https://github.com/denniswjpg/padkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->

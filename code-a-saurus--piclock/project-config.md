---
trigger: always_on
description: _This document is LLM-generated._
---

# AGENTS.md — PiClock

_This document is LLM-generated._

Guidance for AI agents (and humans) working in this repo. Read this before making changes.

## What this is

An always-on digital bedroom clock built from a **Raspberry Pi Zero W** (or **Zero 2 W**)
driving an **Adafruit 1.2" 4-Digit 7-Segment Display with I2C Backpack** (HT16K33). It lives in a
3D-printed housing. Design goals, in priority order:

1. **Never needs to be reset.** It boots, finds the network, and shows the correct local
   time with no human interaction — ever.
2. **Always correct across DST.** Spring-forward / fall-back transitions happen
   automatically with no code change and no manual adjustment.
3. **Survives power loss.** Unplug it, plug it back in; it recovers on its own.
4. Quiet and legible in a dark bedroom (sensible default brightness, non-blinking colon).

## How "never reset / always-correct DST" actually works

The Pi has **no battery-backed RTC**. We do not add one. Correctness comes from:

- **NTP over Wi-Fi** keeps the system clock accurate (`systemd-timesyncd`, on by default
  in Raspberry Pi OS). No network at boot → time is wrong until Wi-Fi associates, then it
  self-corrects. The clock software should tolerate the brief "time not yet synced" window
  rather than displaying a bogus time (see Software design).
- **The IANA tz database** handles DST. We set the timezone **once** on the Pi
  (`sudo timedatectl set-timezone America/New_York` or wherever it lives) and read **local
  time** in Python. `zoneinfo` applies DST rules automatically; OS package updates keep the
  tz rules current. **Never hardcode a UTC offset.** Never compute DST ourselves.

So: NTP for the instant, tzdata for the offset. That combination is the whole reliability
story — keep it intact.

## Hardware

- **Board:** Raspberry Pi Zero W or Zero 2 W, running **Raspberry Pi OS Lite** (headless, no
  desktop). One code path serves both; see the architecture note under Software design re: the
  arch-selected zeroconf pin. The Zero W is armv6 (32-bit OS only); the Zero 2 W is
  armv7l/aarch64 (32- or 64-bit OS).
- **Display:** Adafruit 1.2" 4-digit 7-segment + I2C backpack, HT16K33 controller.
  - I2C default address **`0x70`** (solderable to 0x71–0x77 if changed — check the board).
  - We drive the HT16K33 **directly over I2C with `smbus2`** — no Blinka/Adafruit stack. The
    segment font and the digit/colon RAM offsets live in [piclock/display.py](piclock/display.py),
    which is the canonical reference; that mapping is the thing to get right.
- **Bus:** I2C is enabled on the Pi (`sudo raspi-config nonint do_i2c 0`); it exposes
  `/dev/i2c-1`. Verify the display address with `i2cdetect -y 1` (expect `0x70`).
- Detailed wiring/build instructions live in [HARDWARE_RUNBOOK.md](HARDWARE_RUNBOOK.md).

## Software design

Default to 24-hour time. The code is split so the logic is testable off-Pi and the only
hardware-touching part is isolated:

- [piclock/clock.py](piclock/clock.py) — **pure** time → `DisplayState` (4-char text + colon). No
  I/O. Shows `----` when the clock isn't synced. The colon is **steady by default** (this is
  a bedside clock — no movement at night); an optional 1 Hz blink is config-gated and derived
  from the wall clock, not a tick counter, so it can't drift.
- [piclock/display.py](piclock/display.py) — the HT16K33 `smbus2` driver, plus a pure
  `build_buffer()` (font/colon → 16-byte RAM) and a `NullDisplay` for off-Pi runs. The
  driver skips redundant I2C writes when nothing changed. The hardware is wrapped in
  `ResilientDisplay`: if the display is absent or a connector is bumped, `show()` never
  raises — it logs once and retries the connection every ~10 s, so the service keeps running
  (no crash-loop) and the correct time appears the instant the display reconnects.
- [piclock/timesync.py](piclock/timesync.py) — `is_ntp_synced()` via `timedatectl show -p
  NTPSynchronized`. Any error → treated as not-synced (so we show dashes, never a bad time).
- [piclock/config.py](piclock/config.py) — `Config` dataclass with `PICLOCK_*` env overrides
  (brightness, schedule on/off + times + wake/night brightness, 24h, blink, I2C bus/address,
  tick, and `PICLOCK_DISPLAY=null` for dev). Validates its inputs at construction, including
  that the schedule edges are in daily order.
- [piclock/schedule.py](piclock/schedule.py) — **pure** day/night schedule logic: wall-clock time →
  phase (`off` / `wake` / `night`). No I/O, unit-tested like `clock.py`.
- [piclock/ctl.py](piclock/ctl.py) — optional CLI control: a Unix **datagram** socket
  (`PICLOCK_CTL_SOCKET`; the unit sets `/run/piclock/ctl.sock` via `RuntimeDirectory=`)
  receives one-line commands from [scripts/piclockctl](scripts/piclockctl) (`on`, `off`,
  `brightness 0-15`; the client's `bright`/`dim` are aliases for levels 15/0). A pure
  `parse()` validates them; a daemon thread queues them for the loop, which applies them to
  `DisplayControl` and mirrors them to HomeKit — same override semantics as Siri. The socket
  is mode 0660, group `clockctl` (install.sh creates the group and adds `clocker` to it so
  the app can chgrp its own socket; humans join the group to use `piclockctl`). Startup is
  fail-soft like HomeKit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [code-a-saurus/PiClock](https://github.com/code-a-saurus/PiClock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

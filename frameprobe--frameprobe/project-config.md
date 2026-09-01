---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an end-to-end display latency measurement tool. It measures the time from a mouse click to visible screen change using custom hardware (photodiode + RP2040 microcontroller). The system has three layers: firmware, host software, and hardware.

## Architecture

**Measurement flow:** The RP2040 sends a USB HID mouse click → the click causes a brightness change in the application on the monitor under test (any app works if the brightness difference is big enough) → photodiode detects the change → ADC samples are sent over serial → `main.py` logs to CSV → `analyze.py` calculates latency.

### Firmware (`arduino/arduino.ino`)
- Board-agnostic within arduino-pico: it needs an ADC input on `A1` (GP27), USB HID, and a NeoPixel status LED. Default FQBN is `rp2040:rp2040:waveshare_rp2040_zero` (the PCB's module); the QT Py perfboard build is `FQBN=rp2040:rp2040:adafruit_qtpy ./flash_rp2040.sh`. `NEOPIXEL_POWER` (a switched NeoPixel rail, QT Py only) is `#ifdef`-guarded, but `PIN_NEOPIXEL` is not — variants without a NeoPixel (`generic`, `rpipico`) don't define it and won't compile as-is
- `analogReadResolution(14)` on pin A1 reads the photodiode via a transimpedance amplifier (fab board: VBPW34S + TLV9061; perfboard: BPW34 + TLC271IP). The RP2040 SAR is 12-bit, so the 14-bit range is upscaled — the extra bits add range, not resolution
- Dark baseline differs per sensor: ~565 ADC counts on the fab PCB, ~950 on the perfboard build — harmless, since `analyze.py`'s midpoint detection measures both reference levels from each row's own samples
- Collects 12,000 ADC samples per test run with 20µs settling delay between reads (~288ms window), sends as CSV-prefixed serial line
- Uses `Mouse16.press()`/`Mouse16.release()` instead of a blocking click — press is non-blocking (~15µs), sampling starts immediately
- Two measurement modes: click (default) and move. Move mode fires a single atomic mouse-move report — 1–32767 HID counts (default 127), direction u/d/l/r with HID Y positive-down (default right). Distances beyond ±127 are never chained (chaining would smear the input edge across multiple 1ms polls); instead the firmware replaces the stock Mouse library with its own HID device (`mouse16.h`/`mouse16.cpp`, class `Mouse16Device`, global instance `Mouse16`) whose descriptor declares 16-bit relative X/Y axes, like real gaming mice — registered at runtime via `USB.registerHIDDevice()` with the same ordering/pidMask as the stock library so USB enumeration is unchanged, and the report ID kept at descriptor byte offset 6 where the core rewrites it. `Mouse16.move()` mirrors the stock `Mouse_::move()` (same mutex/`tud_task`/`HIDReady` guard, one `tud_hid_report`, ~15-20µs), so the CSV `clickTime` field keeps its semantics for both press and move. Because the sketch no longer includes the stock `Mouse.h`, `mouse16.cpp` has to include `<tusb-hid.h>` itself on arduino-pico ≥ 5.5.0 — that release moved the TinyUSB HID class driver out of the prebuilt `libpico.a` into a dummy library, leaving only weak no-op stubs in the core, so without the include the link fails with undefined `tud_hid_n_report`/`tud_hid_n_ready`; the include is version-guarded on `ARDUINO_PICO_MAJOR`/`MINOR` since the library doesn't exist on 5.4.x. The inverse move restores the cursor after the CSV dump + `Serial.flush()` (capture is long over by then; the inter-run delay provides settle time) — running inside `runTest()` means `stop` can never strand the cursor displaced. Settings are RAM-only like interval/clicks. Serial commands: `mc`/`mm` (mode), `x<1-32767>` (distance), `r<u|d|l|r>` (direction), `t` (move test: move + 1s hold + inverse move, for visually checking the distance). The Vulkan color-switcher stays click-only
- 20µs ADC settling delay is critical: without it, the sample-and-hold capacitor doesn't fully discharge, compressing dynamic range (black reads ~1238 instead of ~950)
- `usb_hid_poll_interval = 1` overrides arduino-pico's weak default of 10 — a 10ms HID bInterval would add a uniform 0-10ms host-poll delay to every measurement; 1ms matches a 1000Hz gaming mouse
- The residual 0-1ms poll wait is measured, not guessed: press/move only queues the report, so after firing, `Mouse16.waitDelivered()` busy-waits (`tud_task()` + `tud_hid_ready()` under `USB.mutex` — the mutex is the core's `tud_task()` reentrancy guard, its 1ms IRQ pump try-enters and defers while user code owns it; device-local only, no bus traffic) until the host's IN token actually picked it up, and the elapsed µs go out as the trailing `deliveryTime` CSV field (0 = never delivered, 5ms timeout; `analyze.py` drops such rows). The wait sits *between* the sampling phases, never inside them — `analyze.py` timestamps by index × average sample period, so pumping `tud_task()` inside the sampling loop would skew every crossing; the ≤1ms sample gap right after the press is before the input even reached the host. Clicks stay µs-jittered (measure-don't-align: re-timing the press onto the 1ms SOF grid would re-quantize click phase vs. scanout, the exact correlation the jitter exists to destroy)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frameprobe/frameprobe](https://github.com/frameprobe/frameprobe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->

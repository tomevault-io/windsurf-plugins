---
trigger: always_on
description: Read this file and `USB_CLOCK.md` before changing or flashing the clock.
---

# Smart Weather Clock agent guide

Read this file and `USB_CLOCK.md` before changing or flashing the clock.

## Project goal

Turn the inexpensive Smart Weather Clock into a USB-configurable custom display.
The working baseline is a USB-only ESP8266 firmware: no Wi-Fi portal, no cloud
dependency, and no stored Wi-Fi password. The user wants changes tested on the
connected physical clock until the screen is visually confirmed working.

## Verified reference state (2026-07-17)

- Custom firmware is flashed and running on the physical clock.
- The flashed custom firmware provides six USB-selectable modes: an animated
  cyan LED Face, the usage dashboard, BTC/USD with 24 one-hour candles, a large
  Clock view, a Noto Sans Thai test view showing `สวัสดี`, and a local digital
  Gallery/slideshow.
- Current persisted display settings are title `WWW.PHUD.ME`, city `BANGKOK`,
  brightness `70`, rotation `0`, active-low backlight, and a teal accent.
- The user confirmed that the display works and the blinking is gone.
- The user visually confirmed the Noto Sans Thai test is readable and
  flicker-free.
- Daily reminder alerts are flashed and physically verified: they remain steady
  for about 60 seconds, return to Usage afterward, and do not flicker or show a
  blank frame.
- The user visually confirmed the BTC/USD price and 1H candlestick layout is
  readable, complete, and flicker-free. BTC is RAM-only data supplied by the Mac,
  while choosing the BTC screen persists across collector-triggered resets.
- The flicker fix is important: never redraw the whole panel every second.
  `usb_clock.cpp` redraws the time rectangle only when the minute changes and
  performs a full redraw only after configuration changes.
- Time and usage are RAM-only and must be synchronized again after power
  loss/reset. Opening the CH340 serial port also resets the ESP8266.
- The Gallery stores up to seven 240 × 240 RGB565 photos in the clock's local
  LittleFS partition. Gallery playback settings persist across USB resets so a
  collector update does not stop a running slideshow.
- The clock stores up to eight daily ASCII reminder labels in EEPROM. Reminder
  list/set/delete sessions restore cached time and usage after the USB reset.
- Mac-side cached usage includes the previous complete snapshot so dashboard
  trend arrows survive CH340 resets. An incomplete first-run Claude baseline is
  valid collector state until all three rotating slots have been populated.
- Gallery replacement uploads are staged in a temporary LittleFS file and must
  pass the FNV checksum before the existing slot is replaced.
- The post-audit firmware build was flashed with written-data hash verification;
  after cached state restoration, the user visually confirmed that the physical
  display remained normal.
- LED Face V1 is flashed with written-data hash verification and the firmware
  answers `STATUS` plus `FACE AUTO`. It adds cyan code-rendered eyes, eight
  automatic/manual mood choices, partial-region animation, and Face/mood
  persistence across USB resets. The user visually confirmed the physical Face
  screen looks good.
- V2-A is flashed and its persistent LaunchAgent is active on the Mac. The
  RAM-only `EMOTION` override and `tools/clock_service.py` passed USB protocol
  checks: one stable process owns the serial device, CLI requests use the
  user-only Unix socket without resetting the clock, cached usage/BTC survives,
  and `STATUS` reports Face Auto with `auto_emotion=focus`. The user visually
  confirmed the V2-A activity-driven transitions and animation work correctly.

## Confirmed hardware

- MCU: ESP8266EX, 26 MHz crystal.
- Flash: 4 MB, DIO, 40 MHz.
- USB bridge: CH340/CH341. Normal tools auto-detect its current
  `/dev/cu.usbserial-*` path after the clock moves between USB ports.
- Display: ST7789, 240 x 240, SPI mode 3.
- Display wiring:
  - SCLK: GPIO14
  - MOSI: GPIO13
  - DC: GPIO0
  - RST: GPIO2
  - CS: GPIO15
  - Backlight: GPIO5, PWM, active-low
- The physical screen and all pins above have been validated by the flashed
  custom firmware, not merely inferred from documentation.

## Recovery artifacts: keep private

Before first flashing a device, make a complete 4 MB stock-flash backup and
record its SHA-256. Store both outside the repository. Never commit, upload, or
attach this backup: stock firmware may contain saved Wi-Fi credentials.

Do not erase the flash or restore a stock image unless the user explicitly asks
or recovery is required. Do not reconnect restored stock firmware to a sensitive
network without first assessing its security.

## Important files

- `src/usb_clock.cpp`: custom USB-only firmware and screen renderer.
- `src/thai_greeting_bitmap.h`: compact Noto Sans Thai test bitmap for
  `สวัสดี`; generated from Noto Sans Thai Regular 2.000 under the SIL OFL 1.1.
- `tools/clockctl.py`: Mac USB configuration client.
- `tools/clock_gui.py`: local-only browser control panel for Face moods, screen
  switching, brightness, usage/BTC refresh, reminders, test pattern, gallery
  uploads, and slideshow.
- `tools/clock_service.py`: persistent Mac serial owner, local command/Gallery
  proxy, usage polling host, and V2-A Mac-idle emotion engine.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r14084/swc-digital](https://github.com/r14084/swc-digital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

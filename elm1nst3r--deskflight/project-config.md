---
trigger: always_on
description: Live flight radar on the **Waveshare ESP32-S3-Touch-AMOLED-1.43** (466×466 round
---

# DeskFlight — Claude working notes

Live flight radar on the **Waveshare ESP32-S3-Touch-AMOLED-1.43** (466×466 round
AMOLED, SH8601/CO5300 driver, FT3168 touch). Aircraft are pulled from the public
ADS-B feed and drawn as blips with a sweep line, range rings, heading arrows,
airline IATA codes, flight-level altitude, and a tap-to-inspect popup.

## Build environment

Arduino sketch (**not** PlatformIO — PlatformIO ships IDF 5.1.x which is
incompatible with the Waveshare display driver that needs IDF 5.5 APIs).

- Tool: **Arduino IDE 2.x** (or Maker Workshop)
- Platform: `esp32:esp32 3.3.8`
- Board: `Waveshare ESP32-S3-Touch-AMOLED-1.43`
- **Required board settings (non-negotiable, the device will brick or not boot):**
  - USB CDC On Boot: **Enabled**
  - Partition Scheme: **16M Flash**
  - PSRAM: **Enabled**
- Library: `lvgl 9.2.2` (install via Library Manager)
- All source files live at the **sketch root** (no `src/` directory) — Arduino IDE requirement.

## File layout

```
DeskFlight.ino           main sketch — setup(), loop(), networkTask
amoled.{cpp,h}           Amoled class (esp_lcd wrapper) — displayOn, invertColor, setRotation
low_level_amoled.{c,h}   Waveshare SH8601/CO5300 vendor driver (IDF 5.5)
FT3168.{cpp,h}           touch driver — hardened against I²C glitches
i2c.{c,h}                shared I²C bus (SDA=47, SCL=48) — FT3168 + QMI8658 IMU + PCF85063 RTC
board_config.h           Waveshare pin definitions
lv_conf.h                LVGL config — Montserrat 14/18/30/36 fonts enabled
config.h                 DeskFlight app constants (zoom levels, API hosts, NVS keys)
nvs_manager.{cpp,h}      AppSettings struct + NVS load/save
wifi_manager.{cpp,h}     STA connect + captive portal (sets 3 locations at once)
geocoding.{cpp,h}        location string → lat/lng via Nominatim
flight_api.{cpp,h}       adsb.lol free (plain HTTP) + ADS-B Exchange paid (HTTPS)
aircraft_manager.{cpp,h} aircraft state with dead-reckoning + LERP correction
radar_ui.{cpp,h}         LVGL canvas radar render + tap handlers + airline lookup
settings_carousel.{cpp,h} single-page-at-a-time settings overlay (8 tiles)
location_presets.{cpp,h} DEAD CODE — old overlay, safe to delete
```

## Current state (stable enough to ship as v0.9)

### Display

- **Hardware 90° clockwise rotation** via `Amoled::setRotation(90)` using
  `esp_lcd_panel_swap_xy` + `esp_lcd_panel_mirror`. USB-C sits at the bottom.
  Touch coords transformed in `lvgl_touchpad_read`: `(rx, ry) → (ry, W-1-rx)`.
- **Logical map orientation** (`map_orient` setting, N/E/S/W up) rotates the
  radar contents around CX/CY in software. Composes cleanly with the panel
  rotation above.

### Touch driver (FT3168.cpp)

Hardened over multiple iterations:
- Initialise buffers to zero (was the root cause of phantom touches — stack
  garbage from failed I²C reads was being interpreted as a touch)
- Check I²C return code on every read; any error → no touch
- Validate touch count in low nibble of reg 0x02; reject `> 5`
- Clamp coordinates at panel bounds (was rejecting frames, broke long-press)
- **120 ms hold-grace window** — single I²C glitch during a held finger keeps
  reporting the last valid coordinates so LVGL doesn't perceive a release.
  Without this, LV_EVENT_LONG_PRESSED_REPEAT counters reset every glitch and
  the RECONFIGURE hold never accumulated.

### Settings (8 tiles, single-page-at-a-time rendering)

Overlay creates persistent chrome (CLOSE, page title, `<`/`>` arrows, page
dots) once; `s_content` container gets `lv_obj_clean`'d and rebuilt on each
navigation. **Peak widget count ~25 (~15 KB internal SRAM), down from ~150
(~90 KB) in the original tileview design.** That rewrite fixed the OOM /
StoreProhibited panic that happened any time settings opened while the
network task was fetching.

Tiles in order:
0. AIRCRAFT LABELS — tap cycles NONE / CALLSIGN / FULL DATA
1. MAX AIRCRAFT — tap cycles 5/10/15/20/30/ALL; renders the N closest planes by partial-sort
2. ORIENTATION — tap cycles N/E/S/W up
3. DATA SOURCE — ADSB.LOL FREE / ADS-B PAID
4. PROX ALERT — big circle, tap ON/OFF
5. LOCATION — three slots, tap to switch (states: empty / pending / ready)
6. SETUP — shows current ssid + postcode; orange RECONFIGURE button (hold 2 sec)
7. SLEEP HOURS — wake/sleep ± controls; **call is commented out in loop() pending TZ support**

All event callbacks check `ignore_event()` (700 ms guard after `show()`) so
the tap that opened the overlay can't bleed through to a button at the same
pixel.

### Setup flow

1. First boot or after RECONFIGURE → captive portal AP `FlightRadar-Setup`
2. User types Wi-Fi creds + **3 location strings** (loc1 required, loc2/3 optional) + optional ADS-B Exchange RapidAPI key
3. Save → device reboots → `setup()` geocodes each non-empty preset via Nominatim (shows `"Locating N/3..."` splash per slot)
4. Lands on preset 0, radar starts
5. Settings → LOCATION → tap to switch any time

`networkTask` re-attempts geocoding every 30 s for any preset that still has
empty lat/lng (one preset per tick, polite to Nominatim).

### Flight API

- **Free path**: plain HTTP `http://api.adsb.lol/v2/lat/{lat}/lon/{lon}/dist/{nm}`
  via `WiFiClient` — no TLS, no mbedTLS allocations, no AES decryption pressure.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elm1nst3r/DeskFlight](https://github.com/elm1nst3r/DeskFlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->

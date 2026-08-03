---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

DigiFrame is a source-available (noncommercial — PolyForm Noncommercial 1.0.0, see `LICENSE.md`) Arduino/ESP32-S3 firmware that drives a 64x64 HUB75 LED matrix as a **smart clock**: NTP clock, Open-Meteo weather, a neutral living ambient scene, GIF playback from LittleFS, scrolling messages, typed **special days** (date + type + message → themed celebration; merges the old "party mode"), a Telegram bot, a cloud dashboard over Web Bluetooth + a local web dashboard, optional **Home Assistant integration over MQTT**, and a WiFi setup hotspot with an on-panel QR code. Being repositioned from a personal "gift frame" — keep it generic, no personal/gift references.

## Repository layout

- `firmware/DigiFrame/` — the Arduino sketch: `DigiFrame.ino` + all `.h` tabs + `partitions.csv`. The folder must stay named `DigiFrame` (Arduino requires the sketch dir name to match the `.ino`). Build/flash target this path.
- `firmware/gifs/` + `firmware/tools/make_default_gifs.ps1` — default GIF-pack source + its generator (writes `firmware/DigiFrame/default_gifs.h`).
- `website/` — cloud dashboard PWA (Web Bluetooth). `stl/glass-frame/` — 3D-printable enclosure. `images/` — README assets.
- Docs at root: `README.md`, `CLAUDE.md`, `FLASHING.md`, `BLE_PROTOCOL.md`.

## Build / Flash

Arduino IDE **or** arduino-cli (installed, reuses `%LOCALAPPDATA%\Arduino15`):

```
arduino-cli compile --fqbn esp32:esp32:esp32s3:FlashSize=16M,PSRAM=opi,PartitionScheme=custom --output-dir build firmware/DigiFrame
```

- **Board:** ESP32S3 Dev Module — Flash 16MB, PSRAM "OPI PSRAM", partition scheme **Custom** (`PartitionScheme=custom`), which uses the sketch's `partitions.csv` — a 16 MB layout with **4 MB OTA app slots** (app0/app1) + ~7.9 MB `ffat` data. (The old `fatflash` scheme's 2 MB app got tight after adding NimBLE; `partitions.csv` doubles it for future features.) Arduino IDE reports "% of 16 MB" for Custom, but the real ceiling is the 4 MB app slot. Growing the app slots moved the data partition, so the first flash of this layout wipes LittleFS once (GIFs/config re-seed on next boot).
- **Libraries** (Library Manager): `ESP32 HUB75 LED MATRIX PANEL DMA Display` (mrfaptastic), `Adafruit GFX Library`, `AnimatedGIF` (Larry Bank), `UniversalTelegramBot` (Brian Lough), `ArduinoJson`, `NimBLE-Arduino` (h2zero — the BLE config service), `PubSubClient` (Nick O'Leary — the Home Assistant MQTT client). QR codes use the `espressif__qrcode` component bundled with the ESP32 core (`#include <qrcode.h>` resolves to it — do NOT install the ricmoo "QRCode" library, it gets shadowed).
- **Flashing:** see FLASHING.md. `build/DigiFrame_flash_at_0x0.bin` (compact, flash at 0x0) preserves LittleFS; `build/DigiFrame.ino.merged.bin` (16MB padded) wipes it. App-only reflash lives at 0x10000. Typical:
  ```
  esptool --chip esp32s3 --port COM5 write-flash 0x0 build/DigiFrame_flash_at_0x0.bin        # keep GIFs/config
  esptool --chip esp32s3 --port COM5 write-flash 0x10000 build/DigiFrame.ino.bin              # app only, fastest
  esptool --chip esp32s3 --port COM5 write-flash 0x0 build/DigiFrame.ino.merged.bin           # factory reset (wipes LittleFS)
  ```
- **Default GIF pack:** `firmware/gifs/*.gif` are embedded in the app image via the auto-generated `firmware/DigiFrame/default_gifs.h` (regenerate with `firmware/tools/make_default_gifs.ps1` after changing `firmware/gifs/`) and copied to LittleFS **once** on first boot (`seedDefaultGifs()`, marker `/.gifs_seeded`) — after that they are ordinary files the user can delete from the dashboard, and deletions stick. Additional GIFs are uploaded via the web dashboard (`http://digiframe.local`). Telegram GIF upload was removed intentionally.

There are no tests, linters, or CI. Primary verification is a clean arduino-cli compile.

## Runtime configuration

Compile-time **defaults** live in `config.h` (WiFi SSID/pass, `BOT_TOKEN`, `ALLOWED_CHAT_ID`, timezone, location, brightness, hotspot `AP_SSID`/`AP_PASS`, `CLOUD_SITE_URL` for the setup QR, and `MQTT_*` for Home Assistant — all placeholders, no personal data). At runtime they are overridden by `/config.json` on LittleFS (keys `ssid`, `pass`, `tgToken`, `tgChat`, `bright`, `charMin`, `lat`, `lon`, `tz`, `mqttEn`, `mqttHost`, `mqttPort`, `mqttUser`, `mqttPass`), editable from the web dashboard, BLE, and (for some) Telegram. Weather lat/lon live in fixed `char` buffers (`cfgLat`/`cfgLon`), not `String`, because core 1 writes them while `weatherTask` (core 0) reads. **Special days** persist separately in `/events.json` as `{d,t,m}` = date/type/message (type = `custom`|`birthday`); no default events are seeded. **Note:** any `BOT_TOKEN`/`WIFI_PASS`/`MQTT_PASS` you compile in are sensitive — the shipped defaults are placeholders, keep them that way in commits.

## Architecture

Single translation unit: `DigiFrame.ino` includes ordered `.h` files (order matters — later headers may call earlier ones; forward decls for `handleTelegram()`/`fetchWeather()` sit in `globals.h`). The actual include order in `DigiFrame.ino` is:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manoharc07/DigiFrame](https://github.com/manoharc07/DigiFrame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

---
trigger: always_on
description: Aether is a full air-quality monitor project built around an ESP32-C3, a Sensirion SEN66, a 3.7" GDEY037T03 e-paper display, a local browser UI, a WebUSB flashing page, and the supporting PCB/enclosure design files. Most agent work should focus on `firmware/`, `manifests/`, and `www/`; `hardware/` and `enclosure/` are reference/design assets and should usually only receive small, targeted documentation or file-organization updates.
---

## Project Summary

Aether is a full air-quality monitor project built around an ESP32-C3, a Sensirion SEN66, a 3.7" GDEY037T03 e-paper display, a local browser UI, a WebUSB flashing page, and the supporting PCB/enclosure design files. Most agent work should focus on `firmware/`, `manifests/`, and `www/`; `hardware/` and `enclosure/` are reference/design assets and should usually only receive small, targeted documentation or file-organization updates.

## Repository Layout

```text
AGENTS.md                                  # This repo-wide guide for coding agents
README.md                                  # User-facing project overview
enclosure/                                 # SolidWorks enclosure + assembly files
firmware/
├── aether.yaml                            # Main ESPHome config
├── components/
│   ├── aether_epaper/
│   │   ├── aether_epaper.h                # Display driver, render loop, mode state
│   │   ├── aether_epaper_layout.h         # Shared layout + drawing primitives
│   │   └── fonts/                         # Adafruit GFX font headers + TTF sources
│   └── aether_web_ui/
│       ├── __init__.py                    # ESPHome codegen + HTML inlining
│       ├── aether_web_ui.h                # AsyncWebHandler + JSON/API endpoints
│       ├── aether_web_ui_html.h           # Auto-generated; do not edit directly
│       └── web/                           # Source HTML/CSS/JS for the device UI
manifests/
├── manifest.json                          # Factory flashing manifest
└── ota-manifest.json                      # OTA manifest consumed by firmware updates
www/
├── index.html                             # Combined Quick Start Guide & WebUSB flasher
├── style.css                              # Site styling
└── app.js                                 # Scroll animations and tab logic
hardware/
├── pcb/                                   # KiCad project
├── manufacturing/                         # PCB zip, BOM, centroid exports
└── datasheets/                            # Reference component datasheets
```

## Agent Priorities

- Prioritize `firmware/`, `www/`, and `manifests/`.
- Keep `hardware/` and `enclosure/` changes brief and surgical unless the user explicitly asks for CAD/PCB work.
- Do not hand-edit `firmware/components/aether_web_ui/aether_web_ui_html.h`; edit `web/index.html`, `web/style.css`, and `web/app.js` instead.

## Firmware Overview

### Core Hardware

- **MCU:** ESP32-C3 (`esp32-c3-devkitm-1`), Arduino framework
- **Sensor:** Sensirion SEN66 via I2C on SDA=10, SCL=0, address `0x6B`
- **Display:** GDEY037T03 416x240 black/white e-paper via SPI
  - MOSI=7, SCK=6, CS=5, DC=4, RST=3, BUSY=1
- **Boot button:** GPIO 9, `INPUT_PULLUP`, active low

### Main Firmware Entry Point

`firmware/aether.yaml` wires together:

- SEN66 sensor entities
- the custom `aether_web_ui` external component
- the e-paper render loop
- Wi-Fi / captive portal / improv setup
- ESPHome OTA + HTTP OTA update support
- the persisted temperature unit preference

### Sensor Update and Render Flow

- The SEN66 platform updates every **5 seconds**.
- A YAML `interval` calls `aether::aether_epaper::tick_and_draw(...)` every **500ms**.
- Display and web UI both consume the same nine sensor values:

| YAML sensor ID | Metric      | Web UI key |
| -------------- | ----------- | ---------- |
| `co2`          | CO2         | `co2`      |
| `temp`         | Temperature | `temp`     |
| `rh`           | Humidity    | `rh`       |
| `pm1_0`        | PM1.0       | `pm1`      |
| `pm2_5`        | PM2.5       | `pm25`     |
| `pm4_0`        | PM4.0       | `pm4`      |
| `pm10_0`       | PM10        | `pm10`     |
| `voc_index`    | VOC Index   | `voc`      |
| `nox_index`    | NOx Index   | `nox`      |

When adding or renaming a metric, keep YAML IDs, `aether_web_ui` config keys, Python codegen, C++ setters/fields, display cache state, and web JS rendering aligned.

### Display Path

`firmware/components/aether_epaper/aether_epaper.h` owns display state and rendering. It has four modes:

- `MODE_BOOT` - boot wordmark animation
- `MODE_NORMAL` - main dashboard
- `MODE_INFO` - device information / QR screen
- `MODE_RESET` - factory reset confirmation

The shared layout math lives in `aether_epaper_layout.h`.

### Temperature Unit Preference

Temperature unit is currently a persisted **ESPHome template select**, not a switch:

- Entity ID: `temp_unit_select`
- Options: `"Fahrenheit"` and `"Celsius"`
- Default initial option: Fahrenheit
- `restore_value: true`

Internally, temperatures stay in Celsius. The display converts at render time, and the web UI toggles units through `POST /api/temp_unit?unit=C|F`.

### Web UI Path

`aether_web_ui` registers an `AsyncWebHandler` on the ESPHome web server and serves:

- `GET /` or `/index.html` - the inlined device dashboard

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnriqueNeyra/aether](https://github.com/EnriqueNeyra/aether) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->

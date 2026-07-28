---
trigger: always_on
description: **AquaPi** is an open-source ESPHome-based aquarium controller and monitor for ESP32 devices. It integrates with Home Assistant and supports Atlas Scientific EZO sensor circuits (pH, EC/salinity, ORP, dissolved oxygen, CO2, RTD, HUM) and EZO peristaltic pumps via I2C, plus Dallas DS18B20 temperature probes, water level sensors, and binary/leak sensors.
---

# AquaPi – Copilot Coding Agent Instructions

## Project Overview

**AquaPi** is an open-source ESPHome-based aquarium controller and monitor for ESP32 devices. It integrates with Home Assistant and supports Atlas Scientific EZO sensor circuits (pH, EC/salinity, ORP, dissolved oxygen, CO2, RTD, HUM) and EZO peristaltic pumps via I2C, plus Dallas DS18B20 temperature probes, water level sensors, and binary/leak sensors.

The repository does **not** contain Python, JavaScript, or traditional application code. All firmware is expressed in **ESPHome YAML configuration files**. There are no source code files to compile manually — ESPHome compiles the YAML into C++ firmware.

---

## Repository Layout

```
aquapi_config.yaml          # Main firmware config — the entry point compiled by CI
aquapi_config.factory.yaml  # Factory variant (includes aquapi_config.yaml as a package)
common/                     # Modular YAML packages, each included from the main config
  device_base.yaml          # ESP32 board, ESPHome core, WiFi, OTA, web server, I2C bus
  aquapi.yaml               # Project identity, dashboard import URL, app version
  substitutions.yaml        # All default substitution values (pins, addresses, intervals)
  ezo_ph.yaml               # EZO pH sensor platform + buttons + calibration actions
  ezo_ec.yaml               # EZO EC (salinity/conductivity) sensor
  ezo_orp.yaml              # EZO ORP sensor
  ezo_do.yaml               # EZO dissolved oxygen sensor
  ezo_co2.yaml              # EZO CO2 sensor
  ezo_hum.yaml              # EZO humidity sensor
  ezo_rtd.yaml              # EZO RTD temperature sensor
  ezo_pmp.yaml              # EZO peristaltic pump (white, address 103)
  ezo_pmp_duo.yaml          # Two-pump config (waste/clean)
  ezo_pmp_blue/orange/yellow.yaml  # Individual colour-coded pump variants
  i2c_detection.yaml        # Binary sensors + counter + scan button for I2C bus health
  temperature_dallas.yaml   # Dallas DS18B20 temperature sensor
  temperature_dallas_index.yaml    # Multi-probe Dallas temperature (index-based)
  water_level.yaml          # Optical water-level sensors
  binary.yaml               # Binary/leak sensors
  debug.yaml                # Heap, loop-time, I2C health score diagnostics (optional)
  ota_https.yaml            # HTTPS OTA update support
  ...
blueprints/                 # Home Assistant automation/script blueprints (YAML)
static/                     # Jekyll site source (GitHub Pages / web installer)
assets/                     # Images, logos
archive/                    # Archived / legacy config variants
.github/workflows/
  ci.yml                    # ESPHome build validation (stable, beta, dev)
  publish-firmware.yml      # Builds and uploads firmware on GitHub release
  publish-pages.yml         # Publishes the GitHub Pages installer site
```

---

## How the Configuration System Works

### Packages and Substitutions

`aquapi_config.yaml` is a **pure package manifest** — it just lists `!include` entries under the `packages:` key. Each `common/*.yaml` file is a self-contained ESPHome package that:

1. Declares its own `substitutions:` defaults.
2. Contributes sensor, button, select, text_sensor, api, or other ESPHome components.
3. Shares the I2C bus instance `bus_a` defined in `device_base.yaml`.

To enable or disable an optional module, comment or uncomment its `!include` line in `aquapi_config.yaml`.

### Customising Without Editing Package Files

Override substitutions **at the top level** of your device config — they take precedence over package defaults. Example:

```yaml
# my_device.yaml
substitutions:
  name: "tank-controller"
  update_ph: "30s"
  update_ec: "never"   # Disable EC sensor updates (sensor not installed)
packages:
  device_base: !include common/device_base.yaml
  aquapi: !include common/aquapi.yaml
  ezo_ph: !include common/ezo_ph.yaml
```

### Disabling Sensors That Are Not Installed

If an EZO sensor package is included but the hardware is not connected, ESPHome will emit `[E][ezo.sensor:088]: read error` every update interval. Fix this by setting the relevant update substitution to `"never"`:

```yaml
substitutions:
  update_ph: "never"       # Disables automatic reads; buttons with detection still work
  update_pmp_green: "never"
```

---

## I2C Bus and EZO Address Reference

The I2C bus is defined in `common/device_base.yaml`:
- **SDA:** GPIO21 (`${sdaPin}`)
- **SCL:** GPIO22 (`${sclPin}`)
- **Bus ID:** `bus_a`

| EZO Sensor | Default I2C Address (dec) | Hex   | Substitution Variable |
|-----------|--------------------------|-------|-----------------------|
| EZO DO    | 97                       | 0x61  | `addDO`               |
| EZO ORP   | 98                       | 0x62  | `addORP`              |
| EZO pH    | 99                       | 0x63  | `addPH`               |
| EZO EC    | 100                      | 0x64  | `addEC`               |
| EZO RTD   | 102                      | 0x66  | `addRTD`              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheRealFalseReality/aquapi](https://github.com/TheRealFalseReality/aquapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

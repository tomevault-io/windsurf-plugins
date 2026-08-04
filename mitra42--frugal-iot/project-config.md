---
trigger: always_on
description: A platform for affordable, easily built sensor networks running on ESP32/ESP8266 microcontrollers.
---

# Frugal-IoT Library

A platform for affordable, easily built sensor networks running on ESP32/ESP8266 microcontrollers.
Library version: 0.1.1. MIT licence. Author: Mitra Ardron.

Wiki: https://github.com/mitra42/frugal-iot/wiki
Repo: https://github.com/mitra42/frugal-iot

## Compatibility

Every example `.ino` file must work in **both**:
- **Arduino IDE** — open the `.ino` directly; dependencies installed via Library Manager
- **PlatformIO** — referenced via `lib_deps = Frugal-IoT@^0.1.1` in `platformio.ini`

Each example directory contains a `platform.h` file alongside the `.ino`. This file is
**auto-generated** from the example's `platformio.ini` by running `scripts/prerelease.bash`
(which calls `scripts/generate_platform_h.py`). It converts `-D FLAG=value` build flags into
`#define` statements and wraps board-specific defines in `#ifdef ARDUINO_BOARD_NAME` guards.

- **PlatformIO** reads flags directly from `platformio.ini`; `platform.h` is not used.
- **Arduino IDE** users include `platform.h` at the top of the `.ino` to get the same defines.

Do not hand-edit `platform.h` — regenerate it by re-running `scripts/prerelease.bash` after
changing `platformio.ini`.

## Directory Structure

```
Frugal-IoT/
├── Frugal-IoT.h           # Main include — include this and nothing else in user code
├── src/
│   ├── _settings.h        # Compile-time defaults and guards
│   ├── defaults.h         # Default values for all settings
│   ├── misc.h/cpp         # Shared helpers (e.g. StringF)
│   ├── system/            # Infrastructure: frugal (System_Frugal), wifi, mqtt, ota, power,
│   │                       #   fs, i2c, spi, time, watchdog, base, group, io, message, discovery…
│   ├── sensor/            # One file pair per sensor type (sht, dht, ht, soil, battery, bh1750,
│   │                       #   loadcell, ds18b20, ms5803, ens160aht21, button, analog, float,
│   │                       #   uint16, health, gps, sensor [base class]…)
│   ├── actuator/          # LED, digital output, OLED, LCD (actuator.h is the base class)
│   └── control/           # Logic blocks (hysteresis, logger, logger_fs, blinken, carousel,
│                           #   oled, oled_loramesher, gsheets, control.h [base class]…)
├── examples/              # One subdirectory per example application
│   ├── sht30/             # Temperature + humidity with optional OLED
│   ├── climate/           # Dual-channel hysteresis control (heating + humidifier relays)
│   ├── loadcell/          # HX711 weight scale
│   ├── soil/              # Soil moisture
│   ├── loramesher/        # LoRa mesh gateway + node
│   ├── agri/              # Agricultural multi-sensor
│   ├── all/               # All sensors demo
│   ├── blinken/           # LED blink patterns
│   ├── datalogger/        # LittleFS data logging
│   ├── ensaht/            # ENS160 air quality + AHT21
│   ├── gps/               # GPS location via NMEA serial module + OLED
│   ├── gsheets/           # Google Sheets integration
│   ├── lcd_sht/           # HD44780 LCD showing a remote SHT node's readings over MQTT
│   ├── lilygohigrow/      # Plant watering (LilyGo HiGrow)
│   ├── ms5803/            # MS5803 pressure sensor
│   ├── power/             # Power mode demonstration
│   ├── remotedisplay/     # OLED showing a remote SHT node's readings over MQTT
│   └── sonoff/            # Sonoff relay module
└── test/
```

Each example directory contains a `.ino` file (the application) and a `platform.h` (hardware pin/address overrides).

## Component Architecture

Frugal-IoT uses four component groups managed by `System_Frugal`:

| Group | Class prefix | Purpose |
|-------|-------------|---------|
| `frugal_iot.sensors` | `Sensor_*` | Read hardware, publish values |
| `frugal_iot.actuators` | `Actuator_*` | Drive hardware outputs |
| `frugal_iot.controls` | `Control_*` | Logic: transform/route signals |
| `frugal_iot.system` | `System_*` | Infrastructure (WiFi, MQTT, OTA…) |

All components inherit from `System_Base` — `Sensor` via the intermediate `System_SensorActuator`,
`Actuator` and `Control` directly. `System_Group` is a separate `System_Base` subclass used as a
*container*: `frugal_iot.sensors`, `frugal_iot.actuators`, `frugal_iot.controls` and `frugal_iot.system`
are each a `System_Group` holding a list of components and forwarding `setup()`/`loop()`/`dispatch()`
to each member.

## System_Frugal API

`System_Frugal` is the one global object every application creates:

```cpp
System_Frugal frugal_iot("org", "project", "device_id", "Human Name");
```

| Method | Call order | Notes |
|--------|-----------|-------|
| `configure_battery(pin)` | Before `pre_setup()` | Optional; adds battery sensor |
| `configure_power(type, cycle_ms, wake_ms)` | Before `pre_setup()` | Sets sleep strategy |
| `pre_setup()` | After battery/power, before everything else | Starts serial, reads filesystem config |
| `configure_mqtt(host, user, pass)` | After `pre_setup()` | MQTT broker connection |
| `sensors->add(new Sensor_*(…))` | After `pre_setup()` | Register sensors |
| `actuators->add(new Actuator_*(…))` | After `pre_setup()` | Register actuators |
| `controls->add(new Control_*(…))` | After `pre_setup()` | Register controls |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitra42/frugal-iot](https://github.com/mitra42/frugal-iot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->

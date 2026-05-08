---
trigger: always_on
description: **HB-RF-ETH-ng** is a modernized ESP32 firmware for the HB-RF-ETH network interface board (version 2.x). It connects HomeMatic radio modules (HM-MOD-RPI-PCB, RPI-RF-MOD) to debmatic / piVCCU3 installations over Ethernet. This is a fork of the original HB-RF-ETH firmware by Alexander Reinert, ported to ESP-IDF 5.x by Xerolux (2025).
---

# CLAUDE.md — AI Assistant Guide for HB-RF-ETH-ng

## Project Overview

**HB-RF-ETH-ng** is a modernized ESP32 firmware for the HB-RF-ETH network interface board (version 2.x). It connects HomeMatic radio modules (HM-MOD-RPI-PCB, RPI-RF-MOD) to debmatic / piVCCU3 installations over Ethernet. This is a fork of the original HB-RF-ETH firmware by Alexander Reinert, ported to ESP-IDF 5.x by Xerolux (2025).

- **Current version:** 2.1.11
- **License:** Creative Commons BY-NC-SA 4.0 (non-commercial)
- **MCU:** ESP32 (WROOM-32), 240 MHz, 4 MB Flash, 328 KB RAM
- **Ethernet PHY:** LAN8720A

---

## Repository Structure

```
HB-RF-ETH-ng/
├── src/                    # C++ firmware source files
├── include/                # C++ header files (shared across components)
│   └── pins.h              # GPIO pin definitions (canonical hardware map)
├── lib/                    # Additional libraries (if any)
├── webui/                  # Vue.js 3 web interface
│   ├── src/                # Vue components, stores, locales
│   └── dist/               # Built WebUI assets (generated, embedded in firmware)
├── test/                   # PlatformIO unit tests
├── boards/                 # Custom PlatformIO board definition
│   └── hb-rf-eth-ng.json
├── docs/                   # Reference documentation
│   ├── API.md              # REST API reference
│   ├── TROUBLESHOOTING.md
│   ├── openapi.yaml        # OpenAPI 3.x spec
│   └── ALT_BUILD_WORKFLOWS.md
├── .github/workflows/      # CI/CD pipelines
├── CMakeLists.txt          # ESP-IDF root CMake config
├── platformio.ini          # PlatformIO project config
├── partitions.csv          # Flash partition table
├── sdkconfig.hb-rf-eth-ng  # ESP-IDF SDK configuration
├── sdkconfig.defaults      # SDK config overrides
├── version.txt             # Single source of truth for version
└── CHANGELOG.md            # Auto-generated changelog
```

---

## Build System

### Toolchain

| Tool | Version |
|------|---------|
| PlatformIO | Latest |
| ESP-IDF framework | 5.5.3 (via `framework-espidf@~3.50503.0`) |
| Xtensa GCC toolchain | 14.2.0+20251107 |
| Platform package | `espressif32@^6.13.0` |

### Building the Firmware

```bash
# Full build (auto-builds WebUI first via pre-script)
pio run

# Build and upload to device
pio run --target upload

# Serial monitor (115200 baud)
pio device monitor

# Clean build artifacts
pio run --target clean
```

### WebUI Build (standalone)

The WebUI is built automatically via `build_webui.py` pre-script. To build manually:

```bash
cd webui
npm install
npm run build      # outputs to webui/dist/
```

Built assets are gzip-compressed and embedded into firmware via `board_build.embed_files` in `platformio.ini`.

### Pre-build Python Scripts

These run automatically before firmware compilation:

| Script | Purpose |
|--------|---------|
| `build_webui.py` | Builds the Vue.js WebUI |
| `rename_webui_files.py` | Gzip-compresses WebUI assets |
| `append_version_to_progname.py` | Appends version to output binary name |

---

## Hardware Pin Map (`include/pins.h`)

| Signal | GPIO |
|--------|------|
| HM UART RX | GPIO 35 |
| HM UART TX | GPIO 2 |
| HM I2C SDA | GPIO 18 |
| HM I2C SCL | GPIO 5 |
| HM Reset | GPIO 23 |
| HM Button | GPIO 34 |
| RGB Red LED | GPIO 15 |
| RGB Green LED | GPIO 14 |
| RGB Blue LED | GPIO 12 |
| Status LED | GPIO 4 |
| Power LED | GPIO 16 |
| DCF77 input | GPIO 39 |
| ETH PHY Power | GPIO 13 |
| ETH MDC | GPIO 32 |
| ETH MDIO | GPIO 33 |
| Board Rev sense | ADC1 / GPIO 36 |
| Supply voltage sense | ADC1 / GPIO 37 |

**Never change pin assignments without verifying hardware schematics.**

---

## Firmware Architecture (`src/`)

The firmware runs on FreeRTOS with separate tasks per subsystem. Key source files:

| File | Responsibility |
|------|---------------|
| `main.cpp` | Entry point, task orchestration, hardware init |
| `ethernet.cpp` | Ethernet interface, DNS cache, DHCP/static IP |
| `radiomoduledetector.cpp` | Auto-detects HM radio module type (UART/I2C probe) |
| `radiomoduleconnector.cpp` | UART/I2C bridge to radio module |
| `rawuartudplistener.cpp` | UDP↔UART bridge (the core protocol relay) |
| `webui.cpp` | HTTP server + all REST API endpoint handlers |
| `settings.cpp` | Persistent config via NVS Flash |
| `monitoring.cpp` | CheckMK agent and MQTT monitoring |
| `mqtt_handler.cpp` | MQTT client, reconnect logic, message dispatch |
| `monitoring_api.cpp` | REST endpoints for monitoring config |
| `ntpclient.cpp` | NTP time sync client |
| `ntpserver.cpp` | NTP server for downstream clients |
| `dcf.cpp` | DCF77 radio time decoding |
| `gps.cpp` | GPS-based precise time sync |
| `rtcdriver.cpp` | Hardware RTC driver |
| `systemclock.cpp` | Unified system time management |
| `led.cpp` | RGB + status LED control |
| `pushbuttonhandler.cpp` | Factory reset button logic |
| `updatecheck.cpp` | OTA firmware update checking |
| `ota_config.cpp` | OTA server configuration |
| `mdnsserver.cpp` | mDNS / Bonjour advertisement |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xerolux/HB-RF-ETH-ng](https://github.com/Xerolux/HB-RF-ETH-ng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

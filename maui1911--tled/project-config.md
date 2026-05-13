---
trigger: always_on
description: ESP32-C6 based LED controller using Matter protocol over Thread network. Like WLED, but for Thread.
---

# TLED - Matter-over-Thread LED Controller

## Project Overview
ESP32-C6 based LED controller using Matter protocol over Thread network. Like WLED, but for Thread.

## Current Status: Phase 5 (Production Polish) - In Progress

### What's Working
- **On/Off control** via Home Assistant
- **Brightness control** (0-254 range)
- **HSV color control** - full color picker in HA
- **Thread Router** - device is FTD (Full Thread Device), can route for mesh
- **NVS persistence** - saves power/brightness/color, restores on reboot
- **Factory reset** - hold button on GPIO9 for 5+ seconds
- **Smooth transitions** - 300ms fades on all color/brightness/power changes
- **Built-in effects** - rainbow, breathing, candle, chase (API only for now)
- **Serial configuration** - configure LED count, GPIO, type via USB serial
- **Web installer** - flash firmware from browser (GitHub Pages)
- **Health monitoring** - periodic heap/temperature logging
- **Watchdog** - automatic reboot on hang (configurable timeout)
- **Power-on behavior** - restore/on/off modes via serial config
- **Temperature sensor** - internal chip temp monitoring with warnings
- **Temperature cluster** - chip temp exposed to Home Assistant as sensor entity

### Hardware Configuration
- **Board:** DFRobot Beetle ESP32-C6 (or any ESP32-C6)
- **LED Strip:** WS2812B on **GPIO5** (10 LEDs configured)
- **Button:** GPIO9 (factory reset)
- **Power:** 5V external supply

## Development Commands

### Build & Flash
```bash
# Source environment
source ~/esp/esp-idf/export.sh && source ~/esp/esp-matter/export.sh

# Build
idf.py build

# Flash (keeps commissioning)
idf.py -p /dev/ttyACM0 flash

# Erase and flash (loses commissioning - need to re-pair)
idf.py -p /dev/ttyACM0 erase-flash && idf.py -p /dev/ttyACM0 flash
```

### Monitor Serial Output
```bash
# Using Python (works without TTY)
/home/maui/.espressif/python_env/idf5.4_py3.13_env/bin/python << 'EOF'
import serial, time
ser = serial.Serial('/dev/ttyACM0', 115200, timeout=1)
ser.dtr = False; ser.rts = True; time.sleep(0.1); ser.rts = False  # Reset
start = time.time()
while time.time() - start < 20:
    data = ser.read(1024)
    if data: print(data.decode('utf-8', errors='ignore'), end='', flush=True)
ser.close()
EOF
```

### Get QR Code for Commissioning
After flashing, the QR code URL appears in serial output:
```
https://project-chip.github.io/connectedhomeip/qrcode.html?data=MT%3AY.K9042C00KA0648G00
```

### Releasing a New Version
Use the release script — it handles version bumps, build, firmware copy, commit, tag, push, and GitHub release in one command:
```bash
./release.sh 0.8.0
```

The script will:
1. Verify you're on `main` and pull latest
2. Update version in `CMakeLists.txt`, `manifest.json`, and `index.html`
3. Build firmware and copy to `web-installer/firmware/`
4. Commit, tag, and push
5. Create GitHub release with `tled.bin` attached

## Project Structure
```
~/dev/TLED/
├── main/
│   ├── app_main.cpp        # Matter setup, endpoint creation
│   ├── app_driver.cpp      # LED strip driver, NVS persistence
│   ├── app_driver.h        # Driver interface
│   ├── app_config.h        # Pin definitions, LED count
│   ├── app_nvs_config.h    # Runtime config types/API
│   ├── app_nvs_config.cpp  # Config persistence to NVS
│   └── Kconfig.projbuild   # menuconfig options
├── web-installer/
│   ├── index.html          # ESP Web Tools installer page
│   ├── manifest.json       # Firmware manifest for flashing
│   ├── copy-firmware.sh    # Copy built firmware for testing
│   └── serve-local.sh      # Local dev server
├── .github/workflows/
│   └── build-and-deploy.yml  # CI/CD for firmware + Pages
├── docs/
│   └── master-plan.md      # Full project plan (5 phases)
├── sdkconfig.defaults      # ESP-IDF config (Thread FTD, Matter, etc)
└── partitions.csv          # Flash partition layout
```

## Key Implementation Details

### Matter Device Type
Using `dimmable_light` as base + manually added ColorControl cluster with HSV-only feature.
This avoids XY and ColorTemperature modes that caused issues with Home Assistant.

### Color Control
- HSV mode only (no XY, no ColorTemp)
- Hue: 0-254 (Matter range) → 0-360 degrees
- Saturation: 0-254 → 0-100%
- Brightness via LevelControl cluster: 0-254

### NVS Persistence
Saves to `tled_state` namespace:
- `power` (bool)
- `brightness` (uint8)
- `hue` (uint8)
- `saturation` (uint8)

Restored on boot before Matter attributes are applied.

### Thread Configuration
- `CONFIG_OPENTHREAD_FTD=y` - Full Thread Device (can be router)
- Device automatically becomes router when mesh needs it
- No special configuration needed - just keep it powered

### Temperature Measurement Cluster
- Endpoint 2 exposes ESP32-C6's internal temperature sensor to Home Assistant
- Updates every 5 seconds via dedicated FreeRTOS task
- Temperature in 0.01°C units (Matter spec): 2500 = 25.00°C
- Range: -10°C to 80°C (ESP32-C6 internal sensor limits)
- If sensor fails to initialize, endpoint shows "unavailable"

### Transition API
```c
// Set brightness with custom transition time
app_driver_light_set_brightness_with_transition(handle, brightness, transition_ms);

// Set color with custom transition time

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maui1911/TLED](https://github.com/maui1911/TLED) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

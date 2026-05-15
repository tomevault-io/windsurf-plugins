---
trigger: always_on
description: **eHack** is a Raspberry Pi Pico-based RF/wireless analysis tool supporting SubGHz (CC1101), 2.4GHz (NRF24L01+), IR, RFID/NFC modules, and portable wireless modules.
---

# eHack AI Agent Guidelines

**eHack** is a Raspberry Pi Pico-based RF/wireless analysis tool supporting SubGHz (CC1101), 2.4GHz (NRF24L01+), IR, RFID/NFC modules, and portable wireless modules.

## Architecture Overview

### Project Structure
- **`src/main.cpp`**: Dual-core setup/loop; core 0 handles UI/menu logic, core 1 runs on separate thread via `setup1()/loop1()`
- **`src/functions.h`**: Global state, hardware pin definitions, menu indices, module structs (batteries, RSSI, brute-force states)
- **`src/visualization.h`**: MenuState enum (~50 states), OLED rendering, menu navigation
- **`src/signal_data.h`**: Pre-baked IR command tables for TV/projector bruteforce
- **`lib/DataTransmission/`**: Master/slave wireless communication between main and portable modules
- **`lib/ELECHOUSE_CC1101/`**: SubGHz module driver
- **`platformio.ini`**: PlatformIO config; build with `platformio run -e pico`, upload with `platformio upload -e pico`

### Core Concepts
1. **State Machine UI**: MenuState enum drives nested menu navigation; each menu has corresponding `menuIndex` variable (e.g., `MAINmenuIndex`, `HFmenuIndex`)
2. **Dual SPI**: CC1101 uses SPI1 (pins 10-13), NRF24L01+ uses SPI (pins 20-22)
3. **Global State Storage**: Most module state lives in `functions.h` globals (RF parameters, signal buffers, brute-force flags, battery voltage)
4. **EEPROM Persistence**: Settings and last-used slots saved to EEPROM (see `loadSettings()`, `saveSettings()` patterns in main.cpp)
5. **Hardware Abstraction**: Individual libraries (RF24, PN532, RCSwitch, IRremote) wrapped; communication via serialized commands in DataTransmission

## Key Patterns & Conventions

### Menu Navigation (Cluster: visualization.h + functions.h)
- MenuState enum defines all states; main loop switches on `currentMenu`
- Each menu has a count variable (`hfMenuCount = 5`) and index (`HFmenuIndex`)
- Button input (Up/Down/OK) increments/decrements index, wraps at boundaries
- Transitions use `currentMenu = NEW_STATE; *menuIndex = 0` (reset to top)
- Example: SubGHz menu > HF > HF_SPECTRUM state renders spectrum via `drawSpectrum()`

### RF Module Communication (DataTransmission)
- Master device (main eHack) instantiated with `MASTER_DEVICE=1` in build flags
- Commands are 1-byte enum (e.g., `COMMAND_HF_SPECTRUM = 0x01`)
- Responses include data payload (spectrum values, battery voltage via `COMMAND_BATTERY_VOLTAGE = 0xFF`)
- NRF24L01+ pipes: master transmits 0x11223344EE, slave listens on 0xA1B2C3D4E5

### Hardware Initialization (main.cpp setup())
- Wire (I2C) on pins 0/1, SPI on pins 20-22, SPI1 on pins 10-13
- IR Receiver/Sender: pins 3/2; RFID coil/power: pins 27/15; BLE: pin 18
- Order matters: initialize Wire/SPI before radio modules; `cc1101Init()` after SPI1 setup
- EEPROM must call `EEPROM.begin(512)` before load/save

### Signal Capture & Storage
- Each module (RA/Barrier/IR/RFID) has slot files on flash (last used slot tracked via `findLastUsedSlot*()`)
- Signals stored as raw timing arrays or protocol structs (e.g., IR commands as {protocol, address, cmd} tuples from `signal_data.h`)
- Brute-force loops iterate over `irCommandsTV[]` (132 TV codes hardcoded), build CC1101 packets, transmit with state machine

### Debug Logging
- Define `DEBUG_eHack` at top of functions.h to enable Serial output via `DBG()` macro
- Similarly, `DEBUG_DT` in DataTransmission.h for radio comms logging
- Serial init: `Serial.begin(9600)` in setup (only if DEBUG enabled)

## Critical Developer Workflows

### Build & Upload
```bash
cd /Users/elvin/Documents/PlatformIO/Projects/eHack
platformio run -e pico                    # Compile
platformio upload -e pico --upload-port /dev/ttyACM0  # Flash (adjust port as needed)
```

### Debugging RF Modules
1. Enable `DEBUG_eHack` in functions.h; enable `DEBUG_DT` in DataTransmission.h
2. Compile and upload; open serial monitor at 9600 baud
3. Watch for "Initializing CC1101", "NRF24 init failed", radio command/response traces

### Adding New RF Feature (SubGHz Example)
1. Add MenuState enum entry in visualization.h (e.g., `HF_NEW_FEATURE`)
2. Create menu count + index in functions.h (e.g., `newFeatureMenuCount = 2`, `newFeatureMenuIndex`)
3. Add button handler in main loop; case on `currentMenu == HF_NEW_FEATURE`
4. Create drawing function in visualization.h (render via `oled.*` API)
5. Implement RF logic using CC1101 driver (e.g., `ELECHOUSE_CC1101::setFreq(433.92)`)
6. If portable module support: add command byte in DataTransmission.h, serialize data, transmit via `communication.send()`

### Modifying Global State (Battery, RSSI, etc.)
- Battery: update `voltageHistory[]` every `BATTERY_CHECK_INTERVAL` (10s); check charging via `isCharging` flag
- RSSI (SubGHz): `rssiBuffer[]` sliding window (100ms steps); read from CC1101, apply exponential smoothing
- Always persist critical settings to EEPROM before power-off (battery management, last menu, detected signals)

## Integration Points & Dependencies

### External Libraries (from platformio.ini)
- **GyverOLED + Adafruit_SSD1306**: OLED display (128x64); use `oled.*` API
- **EncButton**: debounced button input (15ms debounce, 300ms hold time)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kelv1n9/eHack](https://github.com/kelv1n9/eHack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->

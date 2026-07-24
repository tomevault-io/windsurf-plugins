---
trigger: always_on
description: **BresserWeatherSensorReceiver** is an Arduino library for receiving and decoding 868 MHz wireless data from Bresser weather sensors using ESP32, ESP8266, or RP2040 microcontrollers with RF transceivers (CC1101, SX1276/RFM95W, SX1262, or LR1121).
---

# GitHub Copilot Instructions for BresserWeatherSensorReceiver

## Project Overview

**BresserWeatherSensorReceiver** is an Arduino library for receiving and decoding 868 MHz wireless data from Bresser weather sensors using ESP32, ESP8266, or RP2040 microcontrollers with RF transceivers (CC1101, SX1276/RFM95W, SX1262, or LR1121).

### Key Responsibilities
- Receive and decode RF messages from multiple Bresser sensor types (5-in-1, 6-in-1, 7-in-1, 8-in-1 weather stations, lightning sensors, water leakage sensors, etc.)
- Support multiple simultaneous sensors with ID filtering and slot management
- Provide post-processing for rain gauge statistics and lightning detection
- Integrate with MQTT, WiFi, OLED displays, SD cards, and Home Assistant

## Architecture

### Core Classes

| Class | File | Purpose |
|-------|------|---------|
| `WeatherSensor` | `WeatherSensor.h/.cpp` | Main receiver class - RF reception, message decoding, multi-sensor management |
| `RainGauge` | `RainGauge.h/.cpp` | Rain statistics - hourly/daily/weekly/monthly accumulation, overflow handling |
| `Lightning` | `Lightning.h/.cpp` | Lightning strike counting, hourly history, post-processing |
| `InitBoard` | `InitBoard.h/.cpp` | Board-specific initialization for 15+ ESP32/ESP8266 variants |

### Key Design Patterns

1. **Union-Based Memory Efficiency**: Sensor data stored in unions to minimize memory footprint
2. **Conditional Compilation**: Features selectable via `WeatherSensorCfg.h` macros
3. **Circular History Buffers**: 60-minute history for rain/lightning with wraparound
4. **Preferences Storage**: Flash-based persistence for sensor configs, rain/lightning history
5. **Slot-Based Multi-Sensor**: Dynamic `std::vector<Sensor>` for managing multiple sensors
6. **Decoder Chain**: Try decoders in order (7-in-1 → 6-in-1 → 5-in-1 → Lightning → Leakage) until success

## Coding Conventions

### Naming Conventions
- **Classes**: `CamelCase` (e.g., `WeatherSensor`, `RainGauge`)
- **Functions/Methods**: `camelCase` (e.g., `getMessage()`, `getData()`)
- **Variables**: `snake_case` (e.g., `sensor_id`, `wind_speed_ms`)
- **Constants/Macros**: `UPPER_SNAKE_CASE` (e.g., `NUM_SENSORS`, `BRESSER_5_IN_1`)
- **Private Members**: No specific prefix convention

### File Organization
```
src/
├── WeatherSensor.h/.cpp         # Core receiver & decoder orchestration
├── WeatherSensorCfg.h           # User configuration (pins, sensors, decoders)
├── WeatherSensorConfig.cpp      # Runtime configuration via JSON
├── WeatherSensorDecoders.cpp    # Protocol decoders (5/6/7-in-1, Lightning, Leakage)
├── WeatherUtils.h/.cpp          # Utility functions (dew point, wind chill, etc.)
├── RainGauge.h/.cpp            # Rain statistics module
├── Lightning.h/.cpp            # Lightning post-processing module
└── InitBoard.h/.cpp            # Board initialization

examples/
├── BresserWeatherSensorBasic/          # Simple receive & print
├── BresserWeatherSensorMQTT/           # MQTT integration
├── BresserWeatherSensorMQTTCustom/     # Advanced MQTT with local copy of library
└── [12 other examples]

test/
└── src/                         # CppUTest unit tests
```

### Header File Structure
Every source file must include:
1. **File header comment block** with:
   - File name and purpose
   - Project URL: `https://github.com/matthias-bs/BresserWeatherSensorReceiver`
   - Credits/references to original work
   - MIT License text
   - Detailed history log with dates and changes
   - Author: Matthias Prinke
2. **Include guards** (not `#pragma once`)
3. **Arduino.h include** for Arduino types
4. **Doxygen-style comments** for public API

### Documentation Standards
- Use `///` for Doxygen comments on functions/classes
- Use `//` for inline explanations
- Add `@brief`, `@param`, `@return` tags for public methods
- Reference rtl_433 project sources when applicable
- Document protocol details and sensor quirks extensively

### Code Style
- **Indentation**: 4 spaces (no tabs)
- **Braces**: Opening brace on same line for functions/classes
- **Line Length**: No strict limit, but keep readable (~100-120 chars preferred)
- **Comments**: Extensive inline documentation explaining RF protocols, sensor behavior, edge cases
- **Debug Output**: Use `log_d()`, `log_i()`, `log_w()`, `log_e()` macros (ESP32/ESP8266)
- **Conditional Features**: Wrap optional features in `#ifdef FEATURE_NAME`

### Error Handling
- Return `DecodeStatus` enum from decoders (e.g., `DECODE_OK`, `DECODE_DIG_ERR`, `DECODE_CHK_ERR`)
- Use validation flags in structs (e.g., `temp_ok`, `humidity_ok`, `wind_ok`)
- Check sensor startup flag to prevent false rain/lightning counts after battery replacement
- Validate CRC/checksums using `lfsr_digest16()` or `crc16()`

## Common Patterns & Idioms

### Conditional Compilation
Use macros in `WeatherSensorCfg.h` to enable/disable features:
```cpp
#define BRESSER_5_IN_1      // Enable 5-in-1 decoder
#define BRESSER_6_IN_1      // Enable 6-in-1 decoder
#define BRESSER_7_IN_1      // Enable 7-in-1 decoder
#define BRESSER_LIGHTNING   // Enable lightning decoder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthias-bs/BresserWeatherSensorReceiver](https://github.com/matthias-bs/BresserWeatherSensorReceiver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

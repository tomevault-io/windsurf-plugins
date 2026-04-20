---
trigger: always_on
description: This is an ESPHome-based integration for Daikin Altherma heat pumps, specifically designed for M5Stack POE (Power over Ethernet) hardware. The project enables Home Assistant to monitor and control Daikin heat pumps via the X10A serial protocol.
---

# Daikin Altherma X10A Connect - Project Guide

## Project Overview

This is an ESPHome-based integration for Daikin Altherma heat pumps, specifically designed for M5Stack POE (Power over Ethernet) hardware. The project enables Home Assistant to monitor and control Daikin heat pumps via the X10A serial protocol.

### Key Features
- Read multiple heat pump parameters (temperatures, operation modes, etc.)
- Control operation modes: Off/Heat/Cool
- Smart grid feature support (4 modes: Free running, Forced off, Recommended on, Forced on)
- Runtime debug mode toggle via Home Assistant switch
- Built on ESPHome for easy configuration and updates
- Uses PoE for reliable power and network connectivity

## Hardware Setup

**Required Components:**
- M5 Stack ESP32 Ethernet Unit with PoE (SKU: U138)
- M5 Stack 4-Relay Unit (SKU: U097)
- M5 Stack ESP32 Downloader Kit (SKU: A105)
- Compatible Daikin Altherma 3 R F heat pump

**Pin Connections:**
- UART: RX=GPIO3, TX=GPIO1, 9600 baud, EVEN parity
- I2C (for relay unit): SDA=GPIO16, SCL=GPIO17
- Ethernet: IP101 PHY on GPIO23/GPIO18/GPIO0/GPIO5

## Project Structure

```
DaikinAlthermaX10AConnect/
├── components/
│   └── daikin_x10a/           # Custom ESPHome component
│       ├── daikin_x10a.cpp    # Main component: UART communication, register
│       │                      #   management, AND conversion logic
│       ├── daikin_x10a.h      # Component header (DaikinX10A class)
│       ├── daikin_package.h   # Packet handling only (buffer, CRC, protocol parsing)
│       ├── register_definitions.cpp  # Register struct definition
│       ├── register_definitions.h    # Register struct + scan interval constant
│       ├── __init__.py        # ESPHome Python codegen (sensor auto-creation)
│       └── manifest.json
├── daikin-x10a.yaml                 # ESPHome configuration (hardware + registers)
└── README.md                  # Detailed documentation
```

### Configuration

**`daikin-x10a.yaml`** is the single configuration file containing:
- Hardware setup (UART, I2C, Ethernet, relays)
- daikin_x10a component initialization with full register list
- Mode and smart grid controls (template selects)
- Debug mode switch

**Register configuration example (in daikin-x10a.yaml):**
```yaml
daikin_x10a:
  id: daikin_comp
  uart_id: daikin_uart
  mode: 1
  registers:
    # mode: 0 = read only, not visible in HA
    - { mode: 0, registryID: 0x61, offset: 2, ConversionID: 105, dataSize: 2, dataType: 1, label: "Refrig. Temp. liquid side (R3T)" }

    # mode: 1 = read AND auto-create sensor in HA
    - { mode: 1, registryID: 0x61, offset: 10, ConversionID: 105, dataSize: 2, dataType: 1, label: "DHW tank temp. (R5T)" }
    # ... sensors with mode=1 automatically appear in Home Assistant!
```

**No manual sensor definitions needed!** Sensors are auto-created for all `mode: 1` registers.

**Visual flow:**
```
Heat Pump X10A Protocol
         ↓
[daikin_x10a component reads via UART]
         ↓
registers: array in daikin-x10a.yaml
  - { mode: 0, label: "..." }  ← Read but not exposed to HA
  - { mode: 1, label: "DHW tank temp. (R5T)" }  ← AUTO-CREATES SENSOR
         ↓
[Home Assistant Entity] (automatically created for mode=1)
```

## Development Guidelines

### ESPHome Configuration
- Config: `daikin-x10a.yaml` (hardware setup, register definitions, controls)
- Uses external_components to pull from this GitHub repository
- Requires secrets file for API encryption key and OTA password

### Custom Component (daikin_x10a)
- Written in C++ following ESPHome component architecture
- `DaikinX10A` class owns all state: registers (`registers_` member), sensors, and conversion logic
- `daikin_package` is a pure packet class: buffer management, CRC, protocol parsing — no register/conversion knowledge
- Conversion functions (`convert_registry_values_`, `convert_one_`, `convertTable*_`) are methods of `DaikinX10A`
- Exposes register values via `get_register_value()` method
- Supports custom register definitions via YAML configuration
- Register definition format: `{ mode, registryID, offset, ConversionID, dataSize, dataType, label }` (+ optional: `unit`, `device_class`, `accuracy_decimals`)

**Dynamic Sensor Registration (IMPLEMENTED):**
- `mode: 0` = Read register but don't expose to HA (data available via `get_register_value()`)
- `mode: 1` = Read register AND automatically create sensor in HA
- No manual template sensor definitions needed for mode=1 registers
- Sensors automatically appear in Home Assistant based on register label

### Code Conventions
- Use ESPHome YAML syntax and conventions
- C++ code follows ESPHome component patterns
- Lambda functions in YAML for sensor value extraction
- Internal switches for relay control (not exposed to HA directly)
- Template selects for user-facing controls

### Relay Logic
**Relay assignments:**
- R1, R2: Heat pump mode control (Off/Cooling/Heating)
- R3, R4: Smart grid control (4-state logic)

**Safety pattern:** Always turn off relays before changing states:
```cpp
id(r1).turn_off();
id(r2).turn_off();
// Then set new state
```

## Important Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dennis3321) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

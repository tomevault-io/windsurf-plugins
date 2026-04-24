---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive Electrochemical Impedance Spectroscopy (EIS) system developed as a Bachelor's Final Project (TFG). The system currently consists of two main components:
- ESP32 firmware for controlling AD5940/AD5941 impedance measurement chips
- MATLAB desktop application acting as a frontend for data acquisition and analysis

The MATLAB application serves as the primary frontend interface with two main server connections:
1. **Real-time measurements**: MQTT connection to server for live ESP32/board data acquisition
2. **Dataset management**: Node-RED connection to server for downloading datasets from InfluxDB databases

Note: A backend server for data management and cloud services will be developed separately for Raspberry Pi deployment using IoTStack containers. Additionally, wired USB connections will be maintained for debugging purposes.

## Development Memories

- Every time we create new files we update the CLAUDE.md file
- **EISAppV2.m created**: Implemented complete path-based MATLAB interface with server integration and ESP32 status monitoring

## Build and Development Commands

### ESP32 Firmware (`esp32_porting_AD594x/`)
```bash
# ESP-IDF workflow
idf.py build           # Compile firmware
idf.py flash          # Flash to ESP32
idf.py monitor        # Serial monitoring

# PlatformIO workflow
pio build             # Build project
pio upload            # Upload firmware
pio device monitor    # Monitor serial output
```


### MATLAB Application (`Matlab_Application/`)
- Open `.mlapp` files in MATLAB App Designer
- Run `EISApp.mlapp` to launch the main application
- Use `readAD5940Data.m` for hardware communication testing

## Architecture Overview

### Component Interaction
```
ESP32 + AD5940/AD5941 ↔ [MQTT via Server] ↔ MATLAB Frontend App ↔ [Node-RED] ↔ InfluxDB
                     ↔ [USB - Debug only] ↔
```

### Key Directories
- `esp32_porting_AD594x/`: ESP32 firmware (C/C++, ESP-IDF/PlatformIO)
- `Matlab_Application/`: Desktop GUI application (MATLAB App Designer)
- `server_integration/`: IoTStack server integration configurations and specifications

## Hardware Configuration

### AD5940/AD5941 Measurement Parameters
- Frequency Range: 0.1 Hz to 200 kHz
- Excitation Voltage: 1-2200 mV peak-to-peak
- DC Bias: ±1.1V range
- TIA Resistor: 200Ω to 160kΩ selectable
- Target Hardware: ESP32-S3 DevKit with AD5940/AD5941 evaluation boards

### Dual Board Pin Configuration
**AD5940 Board:**
- SPI: SCLK=13, MISO=12, MOSI=14
- Control: CS=9, INT=10, RST=11

**AD5941 Board:**
- SPI: SCLK=18, MISO=19, MOSI=23
- Control: CS=4, INT=3, RST=15

### Board Selection System
- **Function Pointer Interface**: `board_interface_t` structure with board-specific implementations
- **Runtime Switching**: `board_select(BOARD_AD5940)` or `board_select(BOARD_AD5941)`
- **MATLAB Integration**: Board selection commands sent via serial/WiFi communication
- **Hardware Isolation**: Separate pin assignments prevent SPI bus conflicts

## Communication Protocols

### Production Architecture
**ESP32 ↔ Server ↔ MATLAB Frontend:**
- **MQTT**: Real-time data streaming from ESP32 to server, consumed by MATLAB
- **Node-RED**: Dataset download from InfluxDB database to MATLAB
- **InfluxDB**: Time-series database for measurement storage
- **JSON Protocol**: Standardized data format across all components

### Development/Debug Architecture
**ESP32 ↔ MATLAB (Direct):**
- Serial USB communication (115200 baud) - **Debug only**
- WiFi TCP/IP connection - **Debug only**
- JSON-based command protocol
- **Board Selection Commands**: Runtime switching between AD5940/AD5941 boards
- **Example Commands**: `"SELECT_BOARD:AD5940"`, `"SELECT_BOARD:AD5941"`


## System Operating Modes

The EIS system supports three distinct operating modes to accommodate different measurement scenarios:

### Mode 1: Real-Time Interactive Measurements
- **Communication**: MATLAB ↔ MQTT ↔ ESP32 (bidirectional)
- **Data Storage**: ESP32 → MQTT → Node-RED → InfluxDB (automatic)
- **User Experience**: Live visualization with immediate parameter control
- **Use Cases**: Interactive experimentation, parameter optimization, immediate analysis
- **Data Persistence**: All measurements automatically stored for later analysis

### Mode 2: Scheduled Autonomous Measurements
- **Programming Phase**: MATLAB → MQTT → ESP32 (schedule configuration)
- **Execution Phase**: ESP32 → MQTT → Node-RED → InfluxDB (autonomous)
- **User Experience**: Program schedule, exit MATLAB, return later for results
- **Use Cases**: Long-term studies (days/weeks), unattended monitoring, battery characterization
- **ESP32 Features**: Internal scheduling, NVS persistence, power management

### Mode 3: Historical Data Analysis
- **Communication**: MATLAB ↔ HTTP ↔ Node-RED ↔ InfluxDB
- **Data Source**: Previously stored measurements from Modes 1 & 2
- **User Experience**: Browse, query, and download historical datasets
- **Use Cases**: Offline analysis, comparative studies, report generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SantiBarrios2002) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

---
trigger: always_on
description: ESP32-Lepton is an ESP-IDF component providing a complete driver implementation for FLIR Lepton 3.x thermal imaging sensors. The component handles SPI-based video streaming (VoSPI), I2C command interface (CCI), and provides a high-level API for integration into ESP32 projects.
---

# GitHub Copilot Instructions for ESP32-Lepton Component

## Project Overview

ESP32-Lepton is an ESP-IDF component providing a complete driver implementation for FLIR Lepton 3.x thermal imaging sensors. The component handles SPI-based video streaming (VoSPI), I2C command interface (CCI), and provides a high-level API for integration into ESP32 projects.

**Key Technologies:**
- Platform: ESP32, ESP32-S3 (ESP-IDF framework)
- Build System: CMake (ESP-IDF component)
- Communication: SPI (VoSPI), I2C (CCI)
- RTOS: FreeRTOS
- DMA: High-speed frame acquisition

---

## Code Style and Formatting

### General Formatting Rules

The project uses **Artistic Style (AStyle)** with a K&R-based configuration (`scripts/astyle.cfg`):

- **Style**: K&R (Kernighan & Ritchie)
- **Indentation**: 4 spaces (no tabs)
- **Line Length**: Maximum 120 characters
- **Braces**: K&R style (opening brace on same line for functions, control structures)
- **Operators**: Space padding around operators: `a = bar((b - c) * a, d--);`
- **Headers**: Space between header and bracket: `if (condition) {`
- **Pointers/References**: Stick to name: `char *pThing`, `char &thing`
- **Conditionals**: Always use braces, even for single-line blocks
- **Switch**: Indent case statements

**Example:**
```cpp
esp_err_t Lepton_Init(Lepton_Handle_t *p_Handle, const Lepton_Config_t *p_Config)
{
    if ((p_Handle == NULL) || (p_Config == NULL)) {
        return ESP_ERR_INVALID_ARG;
    }
    
    esp_err_t Error = initialize_spi_bus(p_Config);
    if (Error != ESP_OK) {
        return Error;
    }
    
    return ESP_OK;
}
```

### Naming Conventions

#### Functions
- **Public API**: `Lepton_FunctionName()` using PascalCase with module prefix
  - Examples: `Lepton_Init()`, `Lepton_CCI_ReadRegister()`, `VoSPI_StartCapture()`
- **Private/Static**: `snake_case` with descriptive names
  - Examples: `init_spi_device()`, `process_segment()`, `calculate_crc()`

#### Variables
- **Local variables**: `PascalCase` — **all** local variables, including booleans and loop-adjacent variables
  - Examples: `RetryCount`, `FrameBuffer`, `Error`, `IsInitialized`, `IsCapturing`
  - Simple single-letter loop counters (`i`, `x`, `y`) are the only exception
- **Pointers**: Prefix with `p_` (e.g., `p_Handle`, `p_Config`, `p_Data`)
- **Global/Static module state**: Prefix with underscore: `_Lepton_State`, `_VoSPI_Context`

#### Constants and Macros
- **All UPPERCASE** with underscores: `LEPTON_WIDTH`, `LEPTON_HEIGHT`, `VOSPI_PACKET_SIZE`
- Module-specific macros should include module prefix: `LEPTON_TELEMETRY_ENABLED`, `CCI_TIMEOUT_MS`

#### Types
- **Structs/Enums**: `Lepton_Description_t` or `ModuleName_Description_t` with `_t` suffix
  - Examples: `Lepton_Handle_t`, `Lepton_Config_t`, `VoSPI_Packet_t`, `CCI_Command_t`
- **Enums**: Use descriptive prefix for values
  - Example: `LEPTON_ERROR_NONE`, `LEPTON_ERROR_TIMEOUT`, `CCI_STATUS_BUSY`

#### File Names
- Header files: `moduleName.h` (e.g., `lepton.h`, `lepton_cci.h`, `vospi.h`)
- Implementation files: `moduleName.cpp` or `moduleName.c`
- Private headers: `Private/internalModule.h`

### Code Organization

#### Directory Structure
```text
ESP32-Lepton/
├── CMakeLists.txt             # Component build configuration
├── idf_component.yml          # Component manifest
├── Kconfig                    # Component configuration options
├── include/                   # Public API headers
│   ├── lepton.h               # Main driver API
│   ├── lepton_cci.h           # CCI high-level interface
│   ├── lepton_capture.h       # Capture task interface
│   ├── CCI/
│   │   └── cci.h              # CCI low-level protocol
│   ├── VoSPI/
│   │   └── vospi.h            # VoSPI interface
│   └── Definitions/           # Type definitions and constants
├── src/                       # Implementation files
│   ├── lepton.cpp
│   ├── lepton_cci.cpp
│   ├── lepton_capture.cpp
│   ├── CCI/
│   │   └── cci.cpp
│   └── VoSPI/
│       └── vospi.cpp
├── docs/                      # AsciiDoc documentation
└── examples/                  # Example applications
```

---

## License and Copyright

### File Headers

**Every source file** (`.cpp`, `.h`, `.c`) must include the following header:

```cpp
/*
 * filename.cpp
 *
 *  Copyright (C) Daniel Kampert, 2026
 *  Website: www.kampis-elektroecke.de
 *  File info: Brief description of the file's purpose.
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program. If not, see <https://www.gnu.org/licenses/>.
 *

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kampi/ESP32-Lepton](https://github.com/Kampi/ESP32-Lepton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

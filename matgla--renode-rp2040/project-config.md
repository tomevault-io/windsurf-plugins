---
trigger: always_on
description: This repository contains a **RP2040 MCU simulation** for the [Renode](https://github.com/renode/renode) emulation framework. It provides a complete simulation of the Raspberry Pi RP2040 microcontroller, including support for various peripherals, multicore execution, and PIO (Programmable I/O) simulation.
---

# Renode RP2040 Simulation - Agent Guide

## Project Overview

This repository contains a **RP2040 MCU simulation** for the [Renode](https://github.com/renode/renode) emulation framework. It provides a complete simulation of the Raspberry Pi RP2040 microcontroller, including support for various peripherals, multicore execution, and PIO (Programmable I/O) simulation.

**Project Status**: Work in progress (WIP) and currently frozen due to lack of time. Some peripherals may contain bugs or incomplete implementations.

### Supported Peripherals

| Peripheral | Status | Notes |
|------------|--------|-------|
| **SIO** | 🟡 Partial | Multicore, dividers partially supported |
| **IRQ** | 🟡 Partial | Propagation from some peripherals implemented |
| **DMA** | 🟢 Full | Ringing and control blocks supported |
| **Clocks** | 🟡 Partial | Mostly stubs with tree propagation |
| **GPIO** | 🟢 Full | Pins manipulation with interrupts |
| **XOSC** | 🟢 Full | Crystal oscillator |
| **ROSC** | 🟢 Full | Ring oscillator |
| **PLL** | 🟢 Full | System and USB PLL |
| **PIO** | 🟡 Partial | External C++ simulator, manual sync needed |
| **UART** | 🟢 Full | PL011-based with DREQ generation |
| **SPI** | 🟡 Partial | Master mode only, clock config not supported |
| **Timers** | 🟡 Partial | Alarms implemented |
| **Watchdog** | 🟢 Full | With LimitTimer tick generator |
| **ADC** | 🟡 Partial | Implemented, RESD files not verified |
| **SSI/XIP** | 🟡 Partial | XIP support/caches stubbed |
| **Resets** | 🟢 Full | Device resetting works |
| **I2C** | 🟢 Full | Master mode, interrupts, DMA support |
| **USB** | 🔴 None | Not implemented |
| **PWM** | 🔴 None | Not implemented |
| **RTC** | 🟢 External | PCF8523 I2C device simulator |

## Technology Stack

- **Primary Language**: C# (for Renode peripherals)
- **PIO Simulator**: C++ (external library)
- **Testing**: Robot Framework + Python + xUnit (C#)
- **Build System**: .NET SDK (for C#), CMake (for PIO simulator)
- **Visualization**: Python (websockets, aiohttp)

## Project Structure

```
.
├── boards/                    # Board configuration files
│   ├── raspberry_pico.repl   # Raspberry Pi Pico board definition
│   ├── initialize_raspberry_pico.resc  # Initialization script
│   └── initialize_custom_board.resc    # Custom board template
├── bootroms/                  # RP2040 bootrom binaries
│   └── rp2040/
├── cores/                     # MCU core definitions
│   ├── rp2040.repl           # RP2040 peripheral description
│   ├── initialize_peripherals.resc     # Peripheral loading script (DLL mode)
│   ├── initialize_peripherals_source.resc # Source-mode peripheral loader
│   └── load_peripherals.py   # DLL loader helper script
├── emulation/                 # C# peripheral implementations
│   ├── peripherals/          # Main peripheral implementations
│   │   ├── adc/              # ADC peripheral
│   │   ├── clocks/           # Clock system (XOSC, ROSC, PLL)
│   │   ├── dma/              # DMA controller
│   │   ├── gpio/             # GPIO and pads
│   │   ├── pio/              # PIO integration
│   │   ├── spi/              # SPI controllers
│   │   ├── timer/            # Timers and watchdog
│   │   ├── uart/             # UART
│   │   └── ...
│   ├── externals/            # External device implementations
│   ├── tests/                # C# unit tests
│   │   └── peripherals/
│   │       ├── i2c/          # I2C unit tests
│   │       └── spi/          # SPI unit tests
│   ├── Peripherals.csproj    # .NET project file
│   ├── Peripherals.Tests.csproj # Unit test project
│   └── emulation.sln         # Visual Studio solution
├── piosim/                    # PIO simulator (C++ shared library)
│   ├── libpiosim.so          # Linux shared library
│   ├── libpiosim.dll         # Windows DLL
│   ├── libpiosim.dylib       # macOS library
│   ├── fetch_piosim.py       # Download script for libraries
│   └── version               # Version specifier
├── tests/                     # Test suite
│   ├── testcases/            # Robot Framework integration tests
│   │   └── i2c/              # I2C peripheral tests
│   ├── unit/                 # Unit tests
│   │   └── i2c/              # I2C unit tests (Python/IronPython)
│   ├── pico-examples/        # Pico SDK examples (cloned)
│   ├── pico_examples_patches/# Patches for pico-examples
│   ├── build_pico_examples.sh# Build script for examples
│   ├── run_tests.py          # Parallel test runner
│   ├── tests.yaml            # Test list
│   └── requirements.txt      # Python dependencies
├── visualization/             # Web-based board visualization
│   ├── visualization.py      # Renode integration script
│   ├── visualization_server.py
│   └── requirements.txt
├── images/                    # Documentation images
├── logs/                      # Log output directory
├── snapshots/                 # Emulation snapshots
├── run_firmware.resc         # Quick firmware execution script
└── setup_venv.sh             # Virtual environment setup
```

## Build and Test Commands

### Prerequisites


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matgla/Renode_RP2040](https://github.com/matgla/Renode_RP2040) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

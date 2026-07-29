---
trigger: always_on
description: `esp-serial-flasher` is a portable C library for flashing or loading applications to RAM of Espressif SoCs from other host microcontrollers. The library supports multiple host platforms (STM32, ESP32, Zephyr OS, Pi Pico, Linux) and target chips (ESP32 family, ESP8266) using various communication interfaces (UART, SPI, USB CDC ACM, SDIO).
---

# Copilot Instructions for esp-serial-flasher

## Repository Overview

`esp-serial-flasher` is a portable C library for flashing or loading applications to RAM of Espressif SoCs from other host microcontrollers. The library supports multiple host platforms (STM32, ESP32, Zephyr OS, Pi Pico, Linux) and target chips (ESP32 family, ESP8266) using various communication interfaces (UART, SPI, USB CDC ACM, SDIO).

**Repository size**: ~500 files, primarily C source code with CMake build system
**Languages**: C (primary), Python (testing), CMake
**Target runtime**: Embedded systems, cross-platform compatibility
**Primary framework**: ESP-IDF component + standalone CMake library

## Build Instructions

### Prerequisites and Environment Setup

**Always install pre-commit hooks before making changes:**

```bash
pip install pre-commit
pre-commit install
pre-commit install -t commit-msg
```

### Basic Library Build (Standalone)

For basic library compilation without platform-specific ports:

```bash
mkdir build && cd build
cmake .. -DPORT=USER_DEFINED
make
```

**Time required**: ~30 seconds
**Postcondition**: Creates `libflasher.a` static library
**Note**: CMake warnings about missing project() call are expected and harmless

### ESP-IDF Component Build

**Requires ESP-IDF environment setup (idf.py must be in PATH)**

```bash
# From any example directory (e.g., examples/esp32_example)
idf.py build
```

**Time required**: 2-5 minutes depending on configuration
**Configuration options**: Set via `idf.py menuconfig` or `-D` flags

### STM32 Build

No pre-built example project is provided for STM32. See [examples/stm32_example/README.md](../examples/stm32_example/README.md) for a guide on generating a CubeMX project for your MCU and integrating esp-serial-flasher.

### Zephyr Build

**Requires Zephyr SDK and west workspace setup**

```bash
export ZEPHYR_SDK_INSTALL_DIR=/path/to/zephyr-sdk
export ZEPHYR_TOOLCHAIN_VARIANT="zephyr"
cd zephyr_workspace/zephyr
west build -p -b esp32_devkitc_wroom/esp32/procpu \
    path/to/examples/zephyr_example \
    -DZEPHYR_EXTRA_MODULES=path/to/esp-serial-flasher
```

**Time required**: 5-15 minutes
**Prerequisites**: Zephyr SDK v0.17.0+, west installed

### Raspberry Pi Pico Build

**Requires Pico SDK and ARM toolchain**

```bash
export PICO_SDK_PATH=/path/to/pico-sdk
cd examples/pi_pico_example
mkdir build && cd build
cmake ..
cmake --build .
```

**Time required**: 1-3 minutes
**Output**: `.uf2` file for flashing to Pico

## Testing Instructions

### Pre-commit Validation

**Always run before committing changes:**

```bash
pre-commit run --all-files
```

**Checks performed**: trailing whitespace, line endings, astyle formatting, ruff linting, conventional commit messages

### QEMU Testing

**Requires compiled QEMU with ESP32 support**

```bash
export QEMU_PATH=/path/to/qemu-system-xtensa
cd test
./run_qemu_test.sh
```

**Time required**: 2-5 minutes
**Prerequisites**: QEMU built with ESP32 machine support

### Hardware-in-the-Loop Testing

**Requires physical hardware setup**

```bash
# Install test dependencies
pip install -r test/requirements_test.txt

# Run tests for specific target
pytest --target=esp32 --port=/dev/ttyUSB0
pytest --target=esp32s3 --port=/dev/ttyUSB1
pytest --target=pi_pico --port=/dev/ttyACM1
pytest --target=zephyr --port=/dev/ttyUSB2
```

**Important**: Some ESP32-S3 tests must run separately:

```bash
# Run these separately, not together
pytest --target=esp32s3 --port=/dev/ttyUSB1 -k 'test_esp32_spi_load_ram_example'
pytest --target=esp32s3 --port=/dev/ttyUSB1 -k 'not test_esp32_spi_load_ram_example'
```

### Build All Examples (ESP-IDF)

**Requires ESP-IDF environment and idf-build-apps**

```bash
pip install -U idf-build-apps
python -m idf_build_apps build -v -p . \
    --recursive \
    --exclude ./examples/binaries \
    --config "sdkconfig.defaults*" \
    --build-dir "build_@w" \
    --check-warnings
```

**Time required**: 10-30 minutes depending on configuration

## Project Layout and Architecture

### Core Library Structure

```
src/                    # Core library implementation
├── esp_loader.c       # Main API implementation
├── protocol_uart.c   # SLIP serial protocol (UART, USB CDC-ACM, …)
├── protocol_spi.c    # SPI communication protocol
├── protocol_sdio.c   # SDIO communication protocol
├── esp_targets.c     # Target chip definitions
├── stubs/            # Per-chip flash stub binaries (auto-generated)
│   ├── esp_stub_esp32.c
│   ├── esp_stub_esp32c2.c
│   └── ...           # one file per supported chip + esp_stubs_table.c
└── md5_hash.c        # Optional MD5 verification

include/               # Public API headers
├── esp_loader.h      # Main API header
├── esp_loader_io.h   # I/O interface definitions
└── serial_io.h       # Serial I/O definitions

port/                  # Platform-specific implementations
├── esp32_port.c      # ESP32 UART implementation
├── esp32_spi_port.c  # ESP32 SPI implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [espressif/esp-serial-flasher](https://github.com/espressif/esp-serial-flasher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

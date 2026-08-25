---
trigger: always_on
description: **esp-flasher-stub** is an embedded firmware project that builds flasher stub binaries for Espressif ESP chips. These stubs are small firmware programs that run on ESP devices to facilitate flash programming via esptool.
---

# ESP Flasher Stub - Copilot Coding Agent Instructions

## Repository Overview

**esp-flasher-stub** is an embedded firmware project that builds flasher stub binaries for Espressif ESP chips. These stubs are small firmware programs that run on ESP devices to facilitate flash programming via esptool.

**Project Status**: Production-ready (since v1.0.0). This project has replaced the deprecated [legacy flasher stub in esptool](https://github.com/espressif/esptool-legacy-flasher-stub/) with a modern, maintainable implementation using CMake and the esp-stub-lib library.

**Documentation**: The project maintains developer-facing documentation in the `docs/` directory:
- `docs/architecture.md` - Firmware architecture, source code structure, modules, and build system internals
- `docs/development-guide.md` - Contributing guidelines, testing, CI/CD, and release process
- `docs/plugin-system.md` - Plugin architecture, FPT design, and guide for adding new plugins

These are linked from the main `README.md`, which serves as the user guide.

**Project Type**: Embedded C firmware with CMake build system
**Languages**: C (firmware), Python (build tools, tests)
**Size**: Small (~11 C source files, ~2000 lines main codebase)
**Target Chips**: esp32, esp32s2, esp32s3, esp32c2, esp32c3, esp32c5, esp32c6, esp32c61, esp32h2, esp32h21, esp32h4, esp32p4-rev1, esp32p4, esp32s31, esp8266
**Build Time**: ~0.5-1.5 seconds per chip, ~10-16 seconds for all chips built by build_all_chips.sh (14 chips)

## Critical Setup Steps (ALWAYS Follow This Order)

### 1. Initialize Submodules (REQUIRED - Do This First)

**ALWAYS** run this before any build operation:
```bash
git submodule update --init --recursive
```

This initializes three submodules:
- `esp-stub-lib/` - Core library for ESP stub functionality (REQUIRED for build)
- `unittests/CMock/` - Mocking framework for tests
- `unittests/Unity/` - Unit testing framework

**Build will fail** without submodules initialized.

### 2. Set Up Python Virtual Environment (REQUIRED)

**ALWAYS** use a virtual environment to avoid dependency conflicts:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install pyelftools
```

**CRITICAL**: The `pyelftools` package is **required** for the build process (used by `tools/elf2json.py` to convert ELF to JSON). The build will fail at the post-build step without it.

**ALWAYS** activate the venv in every terminal session:
```bash
source venv/bin/activate
```

### 3. Install ESP Toolchains (For Firmware Build Only)

For AMD64 Linux machines, use the convenience script:
```bash
mkdir -p toolchains
cd toolchains
../tools/setup_toolchains.sh
cd ..
```

This downloads and extracts three toolchains (takes ~2-5 minutes):
1. `xtensa-esp-elf-15.1.0_20250607` - For esp32, esp32s2, esp32s3 (~120MB download)
2. `xtensa-lx106-elf-gcc8_4_0-esp-2020r3` - For esp8266 (~100MB download)
3. `riscv32-esp-elf-15.1.0_20250607` - For esp32c2, esp32c3, esp32c5, esp32c6, esp32c61, esp32h2, esp32h21, esp32h4, esp32p4-rev1, esp32p4, esp32s31 (~255MB download)

**Note**: Network issues may cause partial downloads. If esp8266 toolchain fails, you can still build other chips.

**ALWAYS** export toolchains before building firmware:
```bash
source ./tools/export_toolchains.sh
```

This script adds toolchain bin directories to PATH. **Must be run in every new terminal session** before building.

## Build Commands

### Host Unit Tests (No Toolchains Required)

**Recommended first step** to validate setup without needing ESP toolchains:
```bash
cd unittests/host
./run-tests.sh
```

**Build Time**: ~10-20 seconds (includes CMake config, mock generation, ninja build, CTest run)
**Dependencies**: gcc, cmake, ninja-build, ruby

This runs native unit tests with CMock/Unity frameworks and validates core functionality (SLIP protocol, NAND plugin, plugin FPT dispatch).

### Build Firmware for Single Chip

**ALWAYS** ensure venv is activated and toolchains exported first:
```bash
source venv/bin/activate
source ./tools/export_toolchains.sh
mkdir -p build
cmake . -B build -G Ninja -DTARGET_CHIP=esp32s2  # Replace with desired chip
ninja -C build
```

**Build Time**: ~0.5-1.5 seconds per chip
**Output Files**:
- `build/src/stub-{chip}.elf` - ELF binary
- `build/{chip}.json` - JSON file with stub data (used by esptool)

**Common Chips**: esp32, esp32s2, esp32s3, esp32c3, esp32c6, esp8266

**Note**: The `--fresh` flag in `tools/build_all_chips.sh` ensures clean builds. When building single chips manually, delete the build directory if switching between chips.

### Build Firmware for All Chips

**ALWAYS** ensure venv is activated and toolchains exported first:
```bash
source venv/bin/activate
source ./tools/export_toolchains.sh
./tools/build_all_chips.sh
```

**Build Time**: ~10-15 seconds for all 14 chips (note: cmake defines 15 total targets, but build script excludes esp32h21)
**Output**: Creates `build-{chip}/` directories for each chip with ELF and JSON files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [espressif/esp-flasher-stub](https://github.com/espressif/esp-flasher-stub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->

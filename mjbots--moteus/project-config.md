---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the complete designs for the moteus brushless servo actuator system, including firmware, hardware designs, and client software. Moteus is a high-performance brushless motor controller with integrated position feedback, designed for robotics applications.

## Build System

The project uses **Bazel** as its primary build system. The specific version of bazel is keyed to the repository, so the built in bazel wrapper **must** be used.  Key commands:

- `tools/bazel build --config=target //:target` - Build firmware

Unit tests can be run with:

- `tools/bazel test --config=host //:host` - Run small (fast) host tests only
- `tools/bazel test --test_size_filters= --test_tag_filters=-manual --config=host //:host` - Run full test suite including simulation regression tests (or equivalently, `./travis-ci.sh`)

The project also exposes a CMake interface for client libraries but does not use CMake for the main build.

## Necessary Ubuntu packages

To run the python unit tests, the following Ubuntu python packages must be installed:

```
sudo apt update
sudo apt install -y python3-build python3-can python3-serial python3-setuptools python3-pyelftools python3-qtpy python3-wheel python3-importlib-metadata python3-scipy python3-usb mypy nodejs
```

The apt packages (numpy, scipy, etc.) are built for the system Python 3.12. If `/usr/local/bin/python3` exists and points to a different Python version, it will shadow the system Python and cause import failures. Remove or rename it:

```
sudo mv /usr/local/bin/python3 /usr/local/bin/python3.bak
```

## Proxy/Offline Builds (Claude Code VM)

**IMPORTANT for Claude Code:** When building in the Claude Code VM, always use the offline build approach below. Do not attempt direct bazel downloads as they will fail due to proxy authentication.

If bazel cannot download dependencies due to proxy authentication issues, use the download script to pre-populate the cache:

```
bash utils/download_bazel_deps.sh
```

**Download time estimate:** The script downloads approximately 650MB of dependencies (LLVM toolchain ~414MB, Boost ~124MB, mbed-os ~88MB, plus smaller packages). Expect 10-15 minutes on a typical connection.

Then run bazel with the cache flags AND sandbox fix:

```
tools/bazel test --config=host //:host --repository_cache=/tmp/repo_cache --distdir=/tmp/bazel_cache --sandbox_writable_path=/dev
```

The `--sandbox_writable_path=/dev` flag is required in the Claude Code VM to work around sandbox restrictions on `/dev/null`. Always include this flag when building in the VM environment.

**If downloads fail:** Do NOT fall back to non-bazel builds or manual `pip install`. The bazel build is the only supported method for this project. If the download script fails:
1. Retry the download script - network issues are often transient
2. Check if specific URLs are blocked and report which ones
3. Ask for help rather than attempting workarounds

## Development Commands

### Command line interaction with devices

The following commands will execute the primary user facing tools
using the code and libraries from the current repository:

- `utils/moteus_tool.py`
- `utils/tview.py`

### Python Client Library

End users will use the following commands, although they are not
recommended for testing new features since they will not use the code
from the repository.

- `pip3 install moteus_gui` - Install GUI tools
- `python3 -m moteus_gui.tview --devices=1` - Launch telemetry viewer
- `python3 -m moteus.moteus_tool --target 1 --calibrate` - Calibrate motor

### Testing and Validation
- Python libraries are located in `lib/python/moteus/`
- C++ libraries are located in `lib/cpp/mjbots/moteus/`
- Firmware tests are in `fw/test/`
- Utility tests are in `utils/test/`

## Architecture

### Firmware (fw/)
- **Core Controller**: `MoteusController` (moteus_controller.h/cc) - Main controller class that orchestrates all subsystems
- **Motor Control**: `BldcServo` (bldc_servo.h/cc) - Brushless DC motor servo implementation with position/velocity/torque control
- **Hardware Abstraction**: `MoteusHw` (moteus_hw.h/cc) - Hardware-specific pin definitions and initialization
- **Communication**: `fdcan.h/cc` - CAN-FD communication protocol implementation
- **Motor Sensing**: `MotorPosition` (motor_position.h) - Encoder and position feedback systems
- **Power Management**: `drv8323.h/cc` - Gate driver control for power MOSFETs

### Client Libraries
- **Python**: `lib/python/moteus/moteus.py` - Main Python client with async support
- **C++**: `lib/cpp/mjbots/moteus/moteus.h` - C++ client library with blocking and async APIs
- **Transport**: Multiple transport layers (CAN-FD, fdcanusb, socketcan)

### Hardware Designs (hw/)
- **controller/**: Legacy r4.11 PCB designs (Eagle CAD)
- **c1/**: Compact controller PCB (KiCad)
- **n1/**: High-current controller PCB (Eagle CAD)
- **x1/**: Latest high-power controller PCB (KiCad)

### Utilities (utils/)
- **moteus_tool.py**: Command-line tool for configuration and diagnostics
- **gui/**: Graphical user interface for real-time monitoring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mjbots/moteus](https://github.com/mjbots/moteus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AirTest Mobile Automation is an object-oriented, multi-process control framework for mobile apps based on the AirTest framework. It enhances AirTest with stability features including connection checks, automatic retries on failure, and automatic reconnection for continuous operation. Primary use case is automating mobile games (particularly Honor of Kings) on Android/iOS devices.

## Core Architecture

### Main Module: `airtest_mobileauto/control.py` (~2150 lines)

This is the primary module containing all core functionality. Key classes and their responsibilities:

#### 1. **Settings** (line 120)
- Global configuration class managing all runtime parameters
- Uses `Config(config_file)` to load settings from YAML files
- Key configuration areas:
  - Node info: `mynode`, `totalnode`, `multiprocessing` for multi-process support
  - Device connection: `LINK_dict`, `BlueStackdir`, `LDPlayerdir`, `MuMudir`, `dockercontain`
  - Time management: Uses UTC+8 (GMT+08:00) timezone via `eastern_eight_tz` for Chinese game cycles
  - Logging: `logger_level`, `logfile_dict` for formatted output with `[MM-DD HH:MM:SS]` timestamps
  - Platform detection: Automatically determines Windows/Linux/macOS control environment

#### 2. **DQWheel** (line 817) - "Tool" utilities
- Multi-process coordination via file system synchronization
- Time management: `timelimit(timekey, limit, init, reset)` for throttling operations
- File operations: `removefile()`, `removefiles()`, synchronization barriers
- Dictionary/position caching: Stores image coordinates in YAML files to reduce repeated template matching
- Temporary file management in `Settings.tmpdir`

#### 3. **deviceOB** (line 1520) - Device Management
- Object-oriented device connection and lifecycle management
- Supported control endpoints:
  - **Windows**: BlueStacks, LDPlayer, MuMu emulators (with start/stop/restart)
  - **Linux**: Docker containers (redroid)
  - **macOS**: iOS devices via tidevice
  - **Cross-platform**: USB Android, WiFi Android, custom commands
- Auto-detects client type from LINK string and Settings configuration
- Key methods:
  - `连接设备(times, timesMax)`: Retry connection logic with device restart on final attempt
  - `重启重连设备()`: Full device restart and reconnection
  - Client detection logic (lines 1550-1568): Determines if BlueStacks/LD/MuMu/Docker/USB/Remote based on Settings and platform

#### 4. **appOB** (line 1939) - APP Management
- App lifecycle: `打开APP()`, `关闭APP()`, `重启APP(sleeptime)`
- APPID validation and correction for regional variations (e.g., mark.via vs mark.via.gp)
- `前台APP()`: Get foreground app (Android only)
- Handles Activity specification: Supports both "com.example.app" and "com.example.app/Activity" formats

#### 5. **TaskManager** (line 2092) - Multi-process Execution
- Orchestrates single vs multi-process task execution
- Uses `multiprocessing.Pool` for parallel execution across `totalnode` processes
- Each process re-reads config and sets node-specific parameters via `Config_mynode()`

### Enhanced AirTest Functions (lines 612-750)

The framework wraps core AirTest functions to add resilience:
- `connect_device()`: Retry wrapper around `connect_device_o()`
- `exists()`, `touch()`, `swipe()`: Auto-retry with connection status checking
- `start_app()`: Special handling for Android monkey errors with `--pct-syskeys 0` parameter
- All wrappers check `connect_status()` and retry once on failure before raising error

### Utility Module: `airtest_mobileauto/pick2yaml.py`

Simple script for migrating dictionary files from `.txt` to `.yaml` format and removing spaces from keys.

### OCR Module: `airtest_mobileauto/ocr.py` (Optional)

Lightweight OCR (Optical Character Recognition) module for text recognition in images, complementing AirTest's image-based recognition. **This is an optional feature** that requires extra dependencies.

**Key Components:**

- **OCREngine class**: EasyOCR (PyTorch) wrapper providing:
  - `recognize_text(img)`: Recognize all text in image, returns list of results with positions and confidence
  - `find_text(img, target_text)`: Find specific text, supports exact/fuzzy matching
  - `find_all_text(img, target_text)`: Find all occurrences of text
  - Auto GPU/CPU detection and fallback

- **Coordinate utilities**:
  - `bbox_to_center()`: Convert bounding box to center point
  - `abs_to_relative()`: Convert absolute pixel coordinates to relative (0-1 range)
  - `relative_to_abs()`: Convert relative to absolute coordinates

- **Result format**: All recognition functions return dictionaries with:
  ```python
  {
      'text': str,                    # Recognized text
      'confidence': float,            # Confidence (0-1)
      'bbox': (x, y, w, h),          # Absolute pixel coordinates
      'center': (cx, cy),            # Center point
      'relative_bbox': (rx, ry, rw, rh),  # Relative coordinates
      'relative_center': (rcx, rcy)       # Relative center
  }
  ```

**Design Rationale:**
- Uses EasyOCR (PyTorch) for best user experience
- CUDA libraries bundled in PyTorch wheel - no system CUDA installation needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cndaqiang/airtest_mobileauto](https://github.com/cndaqiang/airtest_mobileauto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

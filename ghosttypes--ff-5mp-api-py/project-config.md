---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FlashForge Python API is a comprehensive Python library for controlling FlashForge 3D printers. The library provides **dual-protocol** support:
- **HTTP API**: Modern REST-like API for Adventurer 5M/5X and Creator 5 series printers
- **TCP/G-code API**: Legacy protocol supporting all networked FlashForge printers

The architecture is fully async/await throughout and uses Pydantic for type-safe data models.

## Core Architecture

### Client Layer Hierarchy

The library has a layered client architecture:

1. **`flashforge.client.FlashForgeClient`** (Main unified client at `flashforge/client.py`)
   - The primary user-facing API that orchestrates both HTTP and TCP communication
   - Manages HTTP session (aiohttp) for modern API endpoints
   - Contains a `tcp_client` instance for legacy operations
   - Provides 5 control modules:
     - `control`: Movement, LED, filtration, camera operations
     - `job_control`: Print job management
     - `info`: Status and machine information
     - `files`: File operations (upload/download/list)
     - `temp_control`: Temperature settings
   - Automatically detects printer capabilities (is_ad5x, is_pro, is_creator5, is_creator5_pro) based on model

2. **`flashforge.tcp.ff_client.FlashForgeClient`** (TCP high-level client)
   - Extends `FlashForgeTcpClient`
   - Implements G-code/M-code command workflows
   - Used internally by the main client's TCP operations
   - Contains `GCodeController` instance for command execution

3. **`flashforge.tcp.tcp_client.FlashForgeTcpClient`** (TCP low-level client)
   - Base TCP communication layer managing socket connections
   - Handles raw command sending/receiving
   - Maintains keep-alive connections
   - Default port: 8899, timeout: 5.0s

### Module Organization

```
flashforge/
├── client.py                    # Main FlashForgeClient (HTTP + TCP orchestrator)
├── discovery/                   # UDP-based printer discovery
│   └── discovery.py            # FlashForgePrinterDiscovery
├── tcp/                        # TCP/G-code protocol implementation
│   ├── tcp_client.py           # Low-level TCP socket management
│   ├── ff_client.py            # High-level G-code client
│   ├── gcode/                  # G-code command definitions and controller
│   │   ├── gcodes.py           # GCodes enum with all commands
│   │   └── gcode_controller.py # GCodeController for executing commands
│   └── parsers/                # Response parsers for TCP commands
│       ├── temp_info.py        # M105 temperature parsing
│       ├── printer_info.py     # M115 printer info parsing
│       ├── thumbnail_info.py   # M662 thumbnail extraction
│       ├── endstop_status.py   # M119 endstop parsing
│       ├── location_info.py    # M114 position parsing
│       └── print_status.py     # M27 print progress parsing
├── api/                        # HTTP API implementation
│   ├── constants/              # Command and endpoint definitions
│   │   ├── commands.py         # Commands enum
│   │   └── endpoints.py        # Endpoints class
│   ├── controls/               # Control modules (used by main client)
│   │   ├── control.py          # Control class
│   │   ├── job_control.py      # JobControl class
│   │   ├── info.py             # Info class
│   │   ├── files.py            # Files class (named 'files' for user API)
│   │   ├── temp_control.py     # TempControl class
│   │   └── creator5_palette.py # Creator 5 material-station palette (CIEDE2000)
│   ├── network/                # Network utilities
│   │   ├── utils.py            # NetworkUtils for HTTP requests
│   │   └── fnet_code.py        # FNetCode for authentication
│   ├── filament/               # Filament handling
│   └── misc/                   # Utilities (temperature, scientific notation)
└── models/                     # Pydantic models for API responses
    ├── responses.py            # All HTTP response models
    └── machine_info.py         # Machine state and info models
```

### Key Design Patterns

**Dual Protocol Strategy**: HTTP is used for high-level operations (printer status, file listing, job control commands) while TCP/G-code is used for real-time operations (temperature monitoring via M105, print progress via M27, thumbnails via M662).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GhostTypes/ff-5mp-api-py](https://github.com/GhostTypes/ff-5mp-api-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

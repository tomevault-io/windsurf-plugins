---
trigger: always_on
description: This project is for developing applications on the GitHub Universe 2025 hackable conference badge - a custom Pimoroni Tufty 2350 edition. It's an RP2350-based device running MicroPython with a 320x240 TFT color display (pixel-doubled to 160x120 for performance) and custom IR sensors.
---

# Copilot Instructions for Universe 2025 Tufty Badge Development

This project is for developing applications on the GitHub Universe 2025 hackable conference badge - a custom Pimoroni Tufty 2350 edition. It's an RP2350-based device running MicroPython with a 320x240 TFT color display (pixel-doubled to 160x120 for performance) and custom IR sensors.

## Project Structure

```
/
├── badge/                     # Badge firmware and apps (deployed to /system/ on device)
│   ├── main.py               # Main entry point and app launcher
│   ├── secrets.py            # WiFi configuration secrets
│   ├── apps/                 # Application directory
│   │   ├── badge/            # GitHub profile stats viewer
│   │   ├── flappy/           # Flappy Bird style game
│   │   ├── gallery/          # Image gallery viewer
│   │   ├── menu/             # App launcher/menu system
│   │   ├── monapet/          # Virtual pet simulator
│   │   ├── quest/            # IR beacon scavenger hunt
│   │   ├── sketch/           # Drawing app
│   │   └── startup/          # Boot animation
│   └── assets/               # Shared assets (fonts, sprites)
│       ├── fonts/            # Pixel Perfect Fonts (.ppf, .af)
│       └── mona-sprites/     # Mona character sprite sheets
└── README.md
```

## Hardware Specifications

- **Processor**: RP2350 Dual-core ARM Cortex-M33 @ 200MHz
- **Memory**: 512kB SRAM, 16MB QSPI XiP flash
- **Display**: 320x240 full colour IPS display (pixel doubled to 160x120 for performance)
- **Screen dimensions**: WIDTH=160, HEIGHT=120 (logical pixels)
- **Connectivity**: 2.4GHz WiFi and Bluetooth 5
- **Battery**: 1000mAh rechargeable (up to 8 hours runtime)
- **Platform**: MicroPython with custom badgeware library
- **Buttons**: 
  - UP, DOWN, A, B, C (front-facing buttons)
  - HOME (back button - triggers quit_to_launcher to return to menu)
  - RESET, BOOTSEL (hardware buttons)
- **IR**: Receiver and transmitter for beacon hunting and remote control
- **Ports**: USB-C (charging/programming), Qw/ST, SWD
- **GPIO**: 4 GPIO pins + power through-hole solder pads
- **LEDs**: 4-zone backlight (TOP_LEFT, TOP_RIGHT, BOTTOM_LEFT, BOTTOM_RIGHT)
- **Case**: Durable polycarbonate with lanyard fixings
- **Runtime**: MicroPython v1.14-5485 with custom badgeware library and built-in modules (see below)

## App Development Guidelines

### Required App Structure

Every app must be in `/system/apps/<app_name>/` (or `badge/apps/<app_name>/` in repository) with:

1. **`__init__.py`** - Main app implementation with required functions:
   ```python
   def init():      # Optional: Called when app starts (for state loading)
   def update():    # Required: Called every frame for input handling and rendering
   def on_exit():   # Optional: Called when quitting app (for state saving)
   ```

2. **`icon.png`** - App icon (used by menu launcher)
   - **Format**: 24x24 pixel color PNG file
   - **Color space**: RGB with optional transparency
   - **Location**: Must be in the app's root directory

### Built-in Modules ###
The following built in modules are available to the MicroPython code running on the device:

array, binascii, builtins, cmath, collections, errno, gc, hashlib, heapq, io, json, machine, math, micropython, network, os, platform, random, re,select, socket, ssl, struct, sys,time, uctypes, rp2, bluetooth, cryptolib, deflate, framebuf, vfs, lwip, ntptime, mip, badgeware,picovector, pimoroni, pimoroni_i2c, qrcode, st7789, powman, board, boot, datetime, ezwifi, pcf85063a, qwstpad, cppmem, adcfft, aioble, asyncio, uasyncio, requests, urequests, urllib, webrepl, websocket, umqtt, ulab, aye_arr, breakout_as7262, breakout_as7343, breakout_bh1745, breakout_bme280, breakout_bme68x, breakout_bme69x, breakout_bmp280, breakout_dotmatrix, breakout_encoder, breakout_encoder_wheel, breakout_icp10125, breakout_ioexpander, breakout_ltr559, breakout_matrix11x7, breakout_mics6814, breakout_msa301, breakout_paa5100, breakout_pmw3901, breakout_potentiometer, breakout_rgbmatrix5x5, breakout_rtc, breakout_scd41, breakout_sgp30, breakout_trackball, breakout_vl53l5cx

### Core Imports and Setup

```python
from badgeware import screen, Image, PixelFont, SpriteSheet, io, brushes, shapes, run, Matrix

# Standard color definitions (RGB values)
BACKGROUND = brushes.color(r, g, b)
FOREGROUND = brushes.color(r, g, b, alpha)  # alpha is optional (0-255)
HIGHLIGHT = brushes.color(r, g, b)

# Set up font - loads from /system/assets/fonts/
screen.font = PixelFont.load("/system/assets/fonts/nope.ppf")

# Enable antialiasing for smooth vector graphics
screen.antialias = Image.X2  # or Image.X4 for higher quality, Image.OFF to disable
```

### State Management Pattern

```python
# Define default state dictionary
state = {
    "key": default_value,
    "count": 0,
    # ... other state variables
}

# Load saved state (merges with defaults) in init()
def init():
    State.load("app_name", state)

# Save state when app exits
def on_exit():
    State.save("app_name", state)

# Update state based on input
def update():

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [badger/home](https://github.com/badger/home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

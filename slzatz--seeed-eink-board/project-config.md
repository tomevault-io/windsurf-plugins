---
trigger: always_on
description: Seeed studios makes a development board called the XIAO ePaper Display Board - EE02 that is designed to drive an e ink spectra 6 13.3 inch display.  The board is based on the ESP32-s3 chip and supports WiFi and Bluetooth connectivity.
---

Seeed studios makes a development board called the XIAO ePaper Display Board - EE02 that is designed to drive an e ink spectra 6 13.3 inch display.  The board is based on the ESP32-s3 chip and supports WiFi and Bluetooth connectivity.

The board is relatively new and there is limited documentation and community support available for it. However, Seeed Studio recently published this documentation: https://wiki.seeedstudio.com/getting_started_with_ee02/#getting-started-with-arduino

There is also a github repository that is trying to do the same thing we are in terms of directly addressing the EE02 XIAO ePaper Display Board at https://github.com/acegallagher/esphome-bigink.

Note that in this repository there is also documentation of the 13.3 inch spectra 6 driver: 13_3_E6_eInk_Display_module_Datasheet.pdf

Seeed provides a web app called the SenseCraft HMI platform to communicate with the e ink display. However. we don't want to go through the WebApp to display images, we want to directly hit the api endpoints that the custom firmware that we build in this repository supports.  If you look at the ~/eink repository on this computer, you will see that I have done something similar with the GooDisplay e ink driver board. We used the GooDisplay web app to reverse engineer the api endpoints and then wrote a python script to hit those endpoints directly but the GooDisplay web app was pretty simple.

This repository now contains custom firmware for the ESP32 on the EEO2 board that runs a web client that generates http requests to our custom image server to display images on the 13.3 inch spectra 6 display.  We also have the capability to put the ESP32 to sleep and have it wake up at intervals to update the display.  The ESP32 wakes up, connects to WiFi, makes a request to the image server to get the image to display, displays the image, and then goes back to sleep.  The image server rotates through the various images in folders that use the MAC address of the EEO@ board.  We can manage the images that each screen displays by adding or deleting images from that screen's image folder (named for its MAC address)

## Custom Firmware Implementation

We have implemented custom Arduino/PlatformIO firmware in the `firmware/` directory:

### Architecture

```
[Home Server]                    [EE02 Board]
image_server.py                  Arduino Firmware
      │                                │
      │ GET /image_packed              │
      │◄──────────────────────────────│ (wake from deep sleep)
      │                                │
      │ Returns packed binary          │
      │ (960KB, pre-dithered)          │
      │──────────────────────────────►│
      │                                │
      │                                │ Display image
      │                                │ (dual-controller SPI)
      │                                │
      │                                │ Deep sleep (15 min default)
      │                                ▼
```

### Display Hardware Details

The 13.3" Spectra 6 display uses dual UC8179 controllers in master/slave configuration:

- **Master (CS=GPIO44)**: Top 600 pixel rows (0-599)
- **Slave (CS=GPIO41)**: Bottom 600 pixel rows (600-1199)
- Both controllers share CLK (GPIO7) and MOSI (GPIO9)

**Other GPIO pins:**
- DC: GPIO10
- Reset: GPIO38
- Busy: GPIO4 (HIGH when busy)
- Power: GPIO43

**Battery monitoring pins (same circuit as EE04 board):**
- Battery ADC: GPIO1 (A0) - voltage divider output
- ADC Enable: GPIO6 (A5) - set HIGH to enable voltage divider, LOW to save power

**Data Format:**
- 4-bit per pixel (2 pixels per byte)
- Total buffer: 960,000 bytes
- Data is transposed during transfer: buffer columns become output rows

### Files

- `firmware/platformio.ini` - PlatformIO project configuration
- `firmware/src/config.h` - WiFi credentials and pin definitions
- `firmware/src/config_manager.h/.cpp` - Persistent configuration storage (NVS)
- `firmware/src/config_server.h/.cpp` - Web-based configuration interface
- `firmware/src/display.h/.cpp` - Spectra 6 display driver (ported from esphome-bigink)
- `firmware/src/main.cpp` - Main loop: WiFi, fetch, display, deep sleep, config mode
- `image_server.py` - Flask server with image rotation and `/image_packed` endpoint
- `.eink_rotation_state.json` - Persisted rotation state (auto-generated, gitignored)

### Runtime Configuration

The server endpoint is configurable at runtime without reflashing:

**To enter configuration mode:**
1. Press the reset button twice quickly (within 2 seconds)
2. The device will either:
   - Connect to your WiFi and show its IP address (open in browser)
   - Or create a WiFi network called "EInk-Setup" (connect and go to http://192.168.4.1)

**Configurable settings:**
- Server host (IP or domain name, e.g., "192.168.86.100" or "myserver.linode.com")
- Server port
- Image endpoint path
- Sleep interval (how often to refresh)

Configuration is stored in NVS (Non-Volatile Storage) and persists across reboots.

### Building and Flashing

1. Install PlatformIO (VSCode extension or CLI)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slzatz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

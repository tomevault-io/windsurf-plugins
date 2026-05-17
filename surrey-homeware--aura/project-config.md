---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Aura ESP32 Weather Widget

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

Aura is an ESP32-based weather widget that runs on ESP32-2432S028R ILI9341 devices with a 2.8" screen (sometimes called "CYD" or Cheap Yellow Display). This is a hardware-specific Arduino project that requires physical ESP32 hardware to build and test.

## CRITICAL LIMITATIONS

**HARDWARE DEPENDENCY**: This project CANNOT be built, tested, or run without the specific ESP32-2432S028R ILI9341 hardware. Do not attempt to build without the hardware.

**NO COMMAND-LINE BUILDS**: This is an Arduino IDE project. There are no Makefiles, package.json, or command-line build tools. All compilation must be done through Arduino IDE.

**NO AUTOMATED TESTING**: There are no unit tests, integration tests, or CI/CD pipelines. All validation must be done manually on the hardware.

## Working Effectively

### Prerequisites
- ESP32-2432S028R ILI9341 device with 2.8" screen (CYD/Cheap Yellow Display hardware)
- Arduino IDE installed and configured
- All required libraries installed in Arduino IDE (see Libraries section below)

### Arduino IDE Configuration
Configure Arduino IDE with these EXACT settings before compiling:
1. Board: "ESP32 Dev Module"
2. Tools -> Partition Scheme: "Huge App (3MB No OTA/1MB SPIFFS)"
3. All other settings can remain at defaults

### Required Libraries Installation
Install these EXACT library versions in Arduino IDE (Sketch -> Include Library -> Manage Libraries):
- ArduinoJson 7.4.1
- HttpClient 2.2.0  
- TFT_eSPI 2.5.43_
- WifiManager 2.0.17
- XPT2046_Touchscreen 1.4
- lvgl 9.2.2

**CRITICAL CONFIGURATION STEP**: After installing libraries, manually copy configuration files:
1. Copy `TFT_eSPI/User_Setup.h` from repository to `~/Documents/Arduino/libraries/TFT_eSPI/User_Setup.h`
2. Copy `lvgl/src/lv_conf.h` from repository to `~/Documents/Arduino/libraries/lvgl/src/lv_conf.h`

### Project Setup
1. Copy the `aura/` folder to `~/Documents/Arduino/aura/`
2. Open `~/Documents/Arduino/aura/weather.ino` in Arduino IDE
3. Verify board and partition scheme settings
4. Connect ESP32 hardware via USB

### Build Process
1. Click "Verify" in Arduino IDE to compile - takes 2-3 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
2. If successful, click "Upload" to flash to device - takes 1-2 minutes. NEVER CANCEL. Set timeout to 5+ minutes.
3. Open Serial Monitor (115200 baud) to view debug output

**Expected Build Time**: 2-3 minutes for verification, 1-2 minutes for upload. NEVER CANCEL builds or uploads.

## Validation

### Hardware Testing Scenarios
After successfully uploading firmware, test these complete scenarios:

1. **Initial Setup Flow**:
   - Power on device
   - Connect to "Aura" WiFi network from phone/laptop
   - Navigate to http://192.168.4.1 in browser
   - Configure WiFi credentials
   - Verify device connects to WiFi and displays weather

2. **Weather Display**:
   - Verify current weather displays with icon and temperature
   - Touch screen to access settings
   - Verify 7-day forecast and hourly forecast views work

3. **Settings Configuration**:
   - Test location search and selection
   - Test brightness adjustment
   - Test language switching (English, Spanish, German, French)
   - Test temperature unit toggle (°C/°F)
   - Test 12/24 hour time format toggle

4. **WiFi Reset**:
   - Test WiFi reset functionality
   - Verify device returns to captive portal mode

### Code Validation
- Always run Arduino IDE's "Verify" before making changes
- Check Serial Monitor output for runtime errors
- Use `extract_unicode_chars.py` when modifying multilingual strings

## Common Tasks

### Key File Locations
```
Repository Structure:
├── aura/
│   ├── weather.ino              # Main Arduino sketch
│   ├── extract_unicode_chars.py # Unicode analysis utility
│   ├── icon_*.c                 # Weather icon assets
│   ├── image_*.c                # Weather image assets
│   └── lv_font_*.c              # LVGL font files
├── TFT_eSPI/
│   └── User_Setup.h             # Display driver configuration
└── lvgl/
    └── src/
        └── lv_conf.h            # LVGL library configuration
```

### Modifying Weather Display
- Main UI creation: `create_ui()` function in weather.ino
- Weather data fetching: `fetch_and_update_weather()` function
- Icon selection: `choose_image()` and `choose_icon()` functions

### Adding New Languages
1. Add language enum to `enum Language`
2. Create new `LocalizedStrings` structure
3. Update language switching logic in settings
4. Run `python3 aura/extract_unicode_chars.py aura/weather.ino` to get required Unicode characters
5. Regenerate LVGL fonts with new character set if needed

### Modifying Display Configuration
- Edit `TFT_eSPI/User_Setup.h` for display driver settings
- Current config: ILI9341_2_DRIVER, 240x320 resolution, specific pin mappings
- Edit `lvgl/src/lv_conf.h` for LVGL library settings
- Current config: 16-bit color depth, ESP32 optimizations

### Font Management
- Custom fonts in `aura/lv_font_*.c` files
- Use LVGL Font Converter tool to generate new fonts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Surrey-Homeware/Aura](https://github.com/Surrey-Homeware/Aura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->

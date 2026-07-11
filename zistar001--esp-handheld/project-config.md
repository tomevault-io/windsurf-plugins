---
trigger: always_on
description: Agent guidance for **ESP-Handheld** firmware — ESP32-S3 handheld with LVGL UI, NES emulator, weather, BLE HID, WiFi audio, and dual-firmware system.
---

# CLAUDE.md

Agent guidance for **ESP-Handheld** firmware — ESP32-S3 handheld with LVGL UI, NES emulator, weather, BLE HID, WiFi audio, and dual-firmware system.

**Project:** `esp_handheld` — ESP-IDF v5.5.4, target `esp32s3`, entry `main/main.c`

## Environment Setup

### Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| ESP-IDF | **v5.5.4** | Must target `esp32s3`. Do NOT use other versions — code relies on v5.5.4 APIs. |
| Python | 3.11+ | Bundled with ESP-IDF installer |
| CMake | ≥3.16 | Bundled with ESP-IDF |
| Ninja | ≥1.10 | Bundled with ESP-IDF |
| Git | Any | For cloning + submodules |

### Install ESP-IDF

**Windows:** Download the ESP-IDF v5.5.4 installer from [Espressif's website](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/get-started/windows-setup.html). Select "ESP32S3" target during installation. After install, use the "ESP-IDF PowerShell" shortcut.

**Linux / macOS:**
```bash
git clone -b v5.5.4 --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh esp32s3
. ./export.sh     # add to .bashrc for convenience
```

**Verify installation:**
```bash
idf.py --version     # Should show v5.5.4
xtensa-esp32s3-elf-gcc --version  # Should show 14.2.0
```

## Build & Flash

### First Clone
```bash
# ⚠️ MUST use --recursive to get submodules (LVGL + esp-wifi-connect)
git clone --recursive https://github.com/zistar001/ESP-Handheld.git
cd ESP-Handheld

# If you forgot --recursive, run this:
git submodule update --init --recursive
```
Submodules: LVGL v8.4 (`components/lvgl`), esp-wifi-connect (`components/esp-wifi-connect`).
CJK font at `components/lvgl/src/font/lv_font_simsun_16_cjk.c` — already in repo.

### Build
```bash
# Must NOT be in MSYS2/Mingw — use PowerShell or cmd.exe
export IDF_COMPONENT_MANAGER=0     # 0 = offline (default), 1 = weather needs zlib
idf.py set-target esp32s3          # first time only (sets target + configures build)
idf.py menuconfig                  # optional: set Weather API Key
idf.py build
```

**Windows parallel build fix:** `idf.py build -- -j2` if `cc1.exe: CreateProcess` error.

### Flash
```bash
# Full (first time):
python -m esptool --chip esp32s3 -p (PORT) -b 921600 write_flash \
  --flash_mode dio --flash_size 16MB --flash_freq 80m \
  0x0 build/bootloader/bootloader.bin \
  0x8000 build/partition_table/partition-table.bin \
  0xd000 build/ota_data_initial.bin \
  0x10000 build/esp_handheld.bin

# App-only (updates):
python -m esptool --chip esp32s3 -p (PORT) -b 921600 write_flash 0x10000 build/esp_handheld.bin

# Monitor:
idf.py -p (PORT) monitor    # 115200 baud, Ctrl+] to exit
```

**CRITICAL:** Without `ota_data_initial.bin` at 0xd000, boots into wrong partition.

### API Key Configuration
```bash
idf.py menuconfig → "ESP-Handheld Configuration" → "Weather API Key"
```
Weather auto-disables if key is empty. Register at [devapi.qweather.com](https://devapi.qweather.com).

## Project Architecture

### Hardware (Reference PCB Rev 2)
ST7789 240×280 SPI LCD + SD card (independent SPI buses) + MAX98357 amp (I2S0 TX) + MSM261 digital mic (I2S1 RX) + LSM6DS3TR-C IMU + AHT20 temp/humidity (shared I2C) + 7 keys + 16MB flash + 8MB PSRAM.

**Pin map in** `main/bsp/bsp_board.h` — edit `BSP_*` defines to adapt to your PCB.

### Partition Layout (16MB)
| Partition | Offset | Size | Content |
|-----------|--------|------|---------|
| factory | 0x10000 | 4MB | ESP-Handheld (this firmware) |
| ota_0 | 0x410000 | 4MB | Retro-Go game system |
| ota_1 | 0x810000 | 4MB | XiaoZhi AI |
| assets | 0xC10000 | ~4MB | SPIFFS resources |

### Code Layout
```
main/
  main.c       — Entry: init sequence + key_handler() dispatch
  bsp/         — Board support (st7789, sd_card, bsp_board, key_driver)
  app/         — App framework (app_manager, launcher, menu)
  modules/     — audio/ imu/ sensor/ weather/ pc_remote/ wifi_manager/
                 settings/ power/ time_sync/ iching/ theme/ spectrum/
                 retro_go/   (rg_adapter: in-process NES via retro-core)
  ui/          — display_driver.c + screens/ + components/
components/
  lvgl/        — LVGL v8.4 (submodule)
  retro-core/  — nofrendo NES C source (in-process)
  retro-go/    — Retro-Go runtime (ota_0 standalone)
  esp-wifi-connect/ — WiFi provisioning (submodule)
```

### Init Sequence (main.c `app_main`)
NVS → Settings/Theme → SD Card → BSP Board (LCD) → LVGL → Audio → Sensors → Battery → BLE HID → Air Mouse → WiFi → Weather/Time → Key Driver → Power Mgmt → App Manager

### App State Machine
```
Launcher ←[START]→ Menu ←[B/START]→ Launcher
                    ↓ [A]        ↑
                 Running (app) ──┘ [B/START]
```

## Key Gotchas

| Issue | Cause | Fix |
|-------|-------|-----|
| MSYS2 detection | ESP-IDF refuses Mingw | Use PowerShell, or `$env:MSYSTEM=''` |
| 1000Hz tick | `CONFIG_FREERTOS_HZ=1000` = 1ms tick | Timing loops run 10x faster than default |
| RGB565 byte-swap | ST7789 needs MSB-first | `CONFIG_LV_COLOR_16_SWAP=y`, raw pixel data must swap |
| LVGL heap 64KB | Screen transitions leak | Always `lv_obj_del(old_scr)` + reset static pointers |
| PSRAM DMA crash | `SPIRAM_USE_MALLOC=y` | Keep `SPIRAM_USE_CAPS_ALLOC=y` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zistar001/ESP-Handheld](https://github.com/zistar001/ESP-Handheld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->

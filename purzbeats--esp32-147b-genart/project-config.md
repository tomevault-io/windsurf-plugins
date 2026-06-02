---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Working notes / resume context for the ESP32-S3 generative-art display project.
Read this first when returning.

> ✅ **DISPLAY WORKS (2026-06-01).** The multi-day black-screen bug was a wrong
> backlight pin: the board is the **1.47B**, which moves the backlight to **GPIO46**
> (the base 1.47 uses GPIO48). With BL on 46 the panel renders. See "RESOLVED" below.
>
> ✅ **GEN-ART APP BUILT (`genart/`, ~91 fps).** Dual-core 16bpp render pipeline at the
> SPI ceiling; effect framework + falling-sand sim with per-run variety; BOOT cycles
> effects. See "The app" section below. **Next: wire the onboard QMI8658 IMU for tilt.**

## Commands

arduino-cli is not on PATH in a fresh shell — prepend machine+user PATH first:

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable('Path','Machine') + ';' + [System.Environment]::GetEnvironmentVariable('Path','User')
$FQBN = "esp32:esp32:esp32s3:PSRAM=disabled,FlashSize=16M,CDCOnBoot=cdc,FlashMode=qio"

arduino-cli compile --fqbn $FQBN .\display_test          # build one sketch (folder name = .ino name)
arduino-cli upload  -p COM3 --fqbn $FQBN .\display_test  # flash over COM3
```

No tests/linters — this is firmware. "Running" = flash, then read the serial
heartbeat (see snippet below) and look at the panel. Each sketch is a separate
Arduino project: `foo/foo.ino` builds/uploads independently.

## Project goal

Generative art on the **Waveshare ESP32-S3-LCD-1.47B** (1.47" ST7789, 172×320).
(Confirmed by silkscreen + the official 1.47B schematic in `docs/`. This is the
"Type B" revision, NOT the base 1.47 — the difference matters: see backlight pin.)
CPU-computed framebuffer art at 30–60 fps; **BOOT button** cycles effects;
RGB LED accent. User asked for "best performance" → we chose **Arduino +
LovyanGFX** (near-C perf, DMA sprites). No GPU: "shaders" = per-pixel CPU render
into a RAM framebuffer, then DMA to the panel.

Performance model: 172×320×2B ≈ 110 KB/frame fits in fast internal SRAM (keep
the framebuffer there, NOT PSRAM — PSRAM is slower for per-pixel writes; use it
for assets). SPI push at 80 MHz ≈ ~11 ms (~90 fps ceiling); real fps is bounded
by pixel-math cost. Use double-buffered LGFX_Sprite + DMA, compute on one core
while the other transfers.

"Tilt control" the user wanted **IS possible** — the **1.47B has an onboard
QMI8658 6-axis IMU on I2C** (confirmed in the 1.47B schematic, `docs/`). The old
note here ("no onboard IMU") was based on the base-1.47 and is WRONG for this board.
Tilt-driven effects need no extra hardware. (Exact IMU_SDA/SCL GPIOs still TBD —
read them off the schematic before wiring it up.) Start with the BOOT button for
effect-switching; add tilt later.

## Hardware (all confirmed)

- ESP32-S3 rev v0.2, dual-core LX7 240 MHz, **8 MB PSRAM, 16 MB flash** (N16R8).
- Native USB-Serial/JTAG: **VID 0x303A / PID 0x1001**, shows up as **COM3**.
  (COM1 is an unrelated built-in port — ignore it.)
- ST7789 172×320 IPS, RGB565.
- **Confirmed pins (1.47B, from the official 1.47B schematic in `docs/`):**
  SCLK 40, MOSI 45, CS 42, DC 41, RST 39, **BL 46 (active HIGH)**, RGB LED (WS2812) 38.
  4-wire SPI (dedicated DC pin). microSD SDMMC: CLK 14, CMD 15, D0 16, D1 18, D2 17, D3 21.
  - ⚠️ **BL is GPIO46 on the 1.47B, NOT 48.** Every base-1.47 source (3D-Box repo,
    ESPP header, Waveshare's own 1.47 demo, the `waveshare_esp32_s3_lcd_147` core
    variant) says BL=48 — that's the *base board* and was the multi-day red herring.
    Backlight defaults OFF (10K gate pulldown), so drive GPIO46 HIGH explicitly.
  - The 1.47B also has an onboard **QMI8658 IMU** (I2C) for tilt; pins TBD from schematic.
- The official esp32 core variant `waveshare_esp32_s3_lcd_147` confirms RGB_LED=38
  (shared with the 1.47B). Do NOT trust that variant for BL — it's 48 (base board).

## Toolchain / environment

- Windows 11, PowerShell. Working dir `C:\code\esp`.
- **arduino-cli 1.5.0** at `C:\Program Files\Arduino CLI\arduino-cli.exe`
  (installed via `winget install ArduinoSA.CLI`). **Gotcha:** not on PATH in a
  fresh shell — prepend machine+user PATH first (see README).
- **esp32:esp32 3.3.8** core; **LovyanGFX 1.2.21** lib (both installed).
- FQBN: `esp32:esp32:esp32s3:PSRAM=disabled,FlashSize=16M,CDCOnBoot=cdc,FlashMode=qio`
- Flashing works perfectly every time (esptool connects, writes, verifies, resets).

### Read serial (no blocking monitor) — handy snippet
```powershell
$port = new-Object System.IO.Ports.SerialPort COM3,115200,None,8,one
$port.ReadTimeout = 1500; $port.Open(); Start-Sleep -Milliseconds 300
$deadline = (Get-Date).AddSeconds(5)
while ((Get-Date) -lt $deadline) { try { $port.ReadLine() } catch {} }
$port.Close()
```

## What WORKS

- ✅ Flash/upload over COM3.
- ✅ **RGB LED** (`rgb_test/`) cycles colors via `rgbLedWrite(38, r,g,b)` — so IO
  control + the pin family are correct. (LED is GRB-ordered; user saw green/red/blue
  for my red/green/blue calls — cosmetic only.)
- ✅ Sketch runs without crashing: `display_test` reaches its loop and prints a
  serial heartbeat (`alive, frame=N`). So `lcd.init()` completes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [purzbeats/esp32-147b-genart](https://github.com/purzbeats/esp32-147b-genart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->

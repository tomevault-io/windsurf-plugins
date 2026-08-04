---
trigger: always_on
description: enables it; names longer than 16 chars will arrive truncated (string[8]).
---

# Darkside PWR — project laws and hard-won facts

Victron power monitor for the truck ("Darkside.Overland" system) on the
ELECROW CrowPanel Advance 3.5". Sibling of the DSODash project and follows
its design language (dark `0x0C1018` bg, tiles `0x111826`, teal/green/amber/
blue accents, montserrat) and its working rules. Read this file before
touching anything — every fact here was verified on hardware or against the
live GX, and several cost real debugging time.

## Machine setup (new computer)

```
brew install arduino-cli gh
arduino-cli config init
arduino-cli config add board_manager.additional_urls https://espressif.github.io/arduino-esp32/package_esp32_index.json
arduino-cli core update-index
arduino-cli core install esp32:esp32@3.3.10     # PINNED — see "Toolchain" below
gh auth login                                   # repo is private
git clone https://github.com/patrickdarke/DarksidePWR
cd DarksidePWR
cp secrets.h.example secrets.h                  # Wi-Fi creds only; config.h has the rest
./build.sh                                      # compile
./build.sh flash                                # compile + flash attached panel
```

`secrets.h` is gitignored — NEVER put real credentials in a tracked file
(this repo's history was verified clean before first push; keep it that
way). Since the config split it holds ONLY Wi-Fi credentials; all GX
addressing/unit-id/display config lives in the committed `config.h`
(#ifndef-guarded, so secrets.h may override any define). The local
secrets.h predates the split and still carries overrides — that's fine,
they win over config.h by design.

Wi-Fi credentials are best set ON DEVICE: gear button (lower right) → Wi-Fi
setup screen (scan/pick/type). Saved to NVS namespace `darkside`, keys
`wifi.ssid`/`wifi.pass`; at boot NVS wins over the secrets.h fallback, and
boot telemetry says which was used: `[pwr] wifi connecting to <ssid> (nvs)`
vs `(secrets.h)`. So a fresh clone builds and provisions with the example's
placeholder credentials untouched. NVS survives reflashes (the app3M_fat9M
partition table keeps the nvs partition); `esptool erase-flash` clears it.

## Hardware (CrowPanel Advance 3.5", board rev V1.2–V1.4)

- ESP32-S3-WROOM-1-N16R8: 16 MB flash, 8 MB **octal** PSRAM (`PSRAM=opi`),
  native USB CDC — enumerates as `/dev/cu.usbmodem*`, no UART bridge.
- Panel: ILI9488, 40 MHz SPI — SCLK 42, MOSI 39, DC 41, CS 40, RST 2;
  `offset_rotation=3` → 480×320 landscape; `invert=true`.
- Touch: GT911 I²C — SDA 15, SCL 16, INT 47, RST 48, addr 0x14 (wired into
  LVGL; used by the gear button → Wi-Fi setup screen).
- Backlight: LEDC PWM on GPIO 38 (20 kHz, 10-bit — backlight.cpp). Percent
  persists in NVS `darkside`/`bright`, floored at `kBacklightMinPct` = 10%
  so a touch-only device can never dim itself to an unreadable screen.
- Expansion headers: J13 = I²C (IO15/16, shared with touch), J15 = UART1
  (IO17/18). Board has TF slot + mic (pins unverified).
- Sound (schematic-verified, V1.4 PDF in the vendor repo):
  - Passive piezo BEEP_5025 on IO8 (net `IO8_BEEP`, SS8050 driver) — THE
    alert path. beeper.cpp drives it with LEDC at ~4 kHz (its resonance);
    chirps play on a short task, never on the LVGL loop.
  - NS4168 I2S mono class-D amp → speaker: BCLK 13, LRC 11, SDATA 12,
    CTRL = IO21 (the vendor "pull 21 low" quirk is this pin; firmware
    still parks it LOW). DEPRECATED for alerts 2026-07-24 — the piezo
    replaced the I2S chime — but the path is verified working (ESP_I2S,
    begin/end per sound; see git history) if music/voice is ever wanted.
  - PDM mic on IO9 CLK / IO10 DATA, shared with the wireless-module SPI
    pads; unused.
  Full-charge chirps: arm after 30 consecutive charging samples, fire once
  when charging stops with SOC >= kChimeSocPct (99, in the .ino), re-arm
  next session. Serial 'B' plays them on demand.
- Case: `case/darksidepwr-case.3mf` (print-ready Bambu Studio project) +
  `.step` (CAD source, Shapr3D export) — owner-designed enclosure for the
  3.5" panel, added 2026-07-25. Photo of the printed case:
  `docs/img/case-photo.jpg` (hero image in README + web flasher; EXIF/GPS
  stripped — keep it that way if ever replaced).
- `LovyanGFX_Driver.h` is ELECROW lesson-03 config, unmodified. Vendor repo:
  `Elecrow-RD/CrowPanel-Advance-3.5-HMI-ESP32-S3-AI-Powered-IPS-Touch-Screen-480x320`
  (revs V1.0 vs V1.2-1.4 have separate example trees).

### Flashing + serial rules (cost a false "black screen" alarm)

- First flash over FACTORY firmware fails the auto-reset ("No serial data
  received") → hold BOOT, tap RST, retry. Once THIS firmware (CDCOnBoot=cdc)
  is on, `./build.sh flash` resets work normally.
- Serial monitor: open the usbmodem port with pyserial **defaults** (DTR/RTS
  asserted). Setting `dtr=False/rts=False` BEFORE open straps the S3 into ROM
  download mode — the screen goes black because the chip is parked in the
  bootloader, not because anything crashed. `script`-wrapped arduino-cli
  monitor does not work (tcgetattr on socket), and killing `arduino-cli
  monitor` loses its buffered output.
- Telemetry: one `[gx] ...` line per 1 Hz poll carries every displayed value —
  verify changes by telemetry, not by eyeball.
- HWCDC backpressure law (cost a fake 6 s "UI stall"): a host that OPENS the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrickdarke/DarksidePWR](https://github.com/patrickdarke/DarksidePWR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->

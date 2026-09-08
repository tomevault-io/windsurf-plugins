---
trigger: always_on
description: Fully reverse engineer the firmware and hardware of the Wobkey Rainy 75 Pro ISO DE keyboard (Telink TLSR9511, RISC-V, Andes D25F core).
---

# Wobkey Rainy 75 Pro ISO DE — Reverse Engineering

## Project Goal

Fully reverse engineer the firmware and hardware of the Wobkey Rainy 75 Pro ISO DE keyboard (Telink TLSR9511, RISC-V, Andes D25F core).

## Key Facts

- **MCU:** Telink TLSR9511 (B91), QFN56, 1MB flash, 256KB SRAM
- **Architecture:** RISC-V RV32IMACF + Andes V5 extensions (XAndesPerf + XAndesCoDense)
- **USB:** VID `0x320F`, PID `0x5055`, 4 HID interfaces
- **Layout:** 75% ISO, 83 keys, Kailh Cocoa Linear switches (Pro), Cherry profile double-shot PBT keycaps
- **Matrix:** 8x16, column-to-row scan, ~500 Hz, 16 col pins + 7 row pins
- **Connectivity:** USB-C (1000Hz/2ms) / BLE 5.0 (500Hz/~8ms) / 2.4GHz (1000Hz/3ms), NKRO all modes
- **Battery:** 7000mAh (2×3500mAh LiPo), ~900h RGB off / ~80h RGB on, charge ~3-4h
- **Firmware:** Evision Semiconductor proprietary platform (120KB with boot header, 115KB body), NOT QMK-based, developer "YKQ"
- **Platform:** Shared across 7+ keyboards (CIDOO, IQUNIX, Ajazz, EPOMAKER) — all VID `0x320F` PID `0x5055`
- **VIA:** V3 (protocol 11), 4 layers, 16 macros, 512B macro buffer
- **RGB:** 83 WS2812 per-key LEDs (no separate underglow) via PSPI MOSI on PB7, DMA ch4, ~6 MHz SPI clock. ZMK firmware drives them with the custom out-of-tree **rainy_rgb** engine (replaces ZMK underglow; 12 effects + an opt-in walker diagnostic + spatial/reactive + functional indicators) — see [docs/rainy-rgb.md](docs/rainy-rgb.md)
- **Ghidra:** 211 functions, 211 named (100%), project uses `RISCV:LE:32:AndeStar_v5`

## Documentation

All technical findings are in `docs/`:
- [docs/zmk-firmware.md](docs/zmk-firmware.md) — ZMK firmware build, BLE HCI driver, board definition, workspace layout
- [docs/rainy-rgb.md](docs/rainy-rgb.md) — rainy_rgb out-of-tree lighting engine: 12 effects + opt-in walker diagnostic, XY calibration, functional indicators (CapsLock/Fn-highlight/battery), controls, build/flash
- [docs/architecture.md](docs/architecture.md) — MCU, USB, HID interfaces, RGB, battery, connection modes
- [docs/gpio-matrix.md](docs/gpio-matrix.md) — GPIO pins, matrix scan, timing, keymap, Fn combos
- [docs/firmware-analysis.md](docs/firmware-analysis.md) — Ghidra, 211 functions, key pipeline, SRAM buffers, decompilation
- [docs/protocols.md](docs/protocols.md) — OTA protocol, VIA protocol, HID probe results
- [docs/hardware-probing.md](docs/hardware-probing.md) — test pads, SWS, logic analyzer, Burning EVK
- [docs/resources.md](docs/resources.md) — downloads, SDK links, reference projects
- [docs/evision-platform.md](docs/evision-platform.md) — Evision firmware platform, sibling keyboards, GearHub protocol
- [docs/wob-driver-analysis.md](docs/wob-driver-analysis.md) — wobwxe.com WOB driver JS analysis, 3 HID protocols, flash address map

## Local Files

```
zmk/                             # Zephyr module — our custom firmware code
  west.yml                       # west manifest (fetches ZMK + hal_telink + mcuboot)
  boards/rainy75/                # HWMv2 board definition (DTS, keymap, defconfig)
  drivers/bluetooth/             # BLE HCI driver (b91_bt.c shim) + deep sleep PM hooks
  drivers/usb/                   # USB DC driver (legacy usb_dc.h API, linked and enabled)
  drivers/led_strip/             # WS2812 LED strip driver (PSPI + DMA, b91_pspi.h registers)
  drivers/sensor/                # Battery ADC driver (SAR ADC sensor + channel scanner)
  drivers/watchdog/              # B91 hardware WDT driver (register-direct, Zephyr wdt API)
  src/mcuboot_confirm.c          # MCUboot image confirmation + WDT safety net
  src/boot_diag.c + .h           # Boot diagnostic: .noinit SRAM buffer + PD7 GPIO heartbeat + PA7 SWS restore
  src/poweroff.c                 # Deep sleep: z_sys_poweroff() — deep retention 64K with GPIO wakeup
  src/flash_mgmt.c               # Custom mcumgr group 64: raw flash erase/write/read/commit + RAM trampoline
  src/rainy_rgb/                 # rainy_rgb lighting engine (color/effects/engine/reactive/overlay/led_map/state/zmk_adapter) — see docs/rainy-rgb.md
  src/behaviors/behavior_rainy_rgb.c  # &rgb keymap behavior (toggle/effect/hue/bright/speed/battery)
  dts/bindings/                  # DTS bindings: b91-usbd / b91-spi-led-strip / b91-battery-adc / b91-watchdog
  lib/liblt_9518_zephyr.a        # BLE controller blob (2.8 MB) — proprietary/NDA, fetched by fetch_ble_blob.sh, gitignored (NOT committed)
conf/                            # build configuration overlays
  app.conf                       # ZMK app config (BLE, USB, mcumgr, WDT, RGB)
  ota-bridge.conf                # OTA bridge config (monolithic, USB+mcumgr+flash_mgmt)
  mcuboot.conf                   # MCUboot bootloader config
  mcuboot.overlay                # MCUboot DTS overlay (disables peripherals)
zmk-src/                         # ZMK upstream (fetched by west)
zephyr/                          # Zephyr upstream (fetched by west)
modules/hal/hal_telink/          # Telink HAL (fetched by west, patched for BT_HCI_B91)
bootloader/mcuboot/              # MCUboot v2.2.0 (fetched by west)
install_zmk.sh                   # stock → ZMK one-command installer (OTA bridge + flash_mgmt)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scholzri/rainy75-zmk](https://github.com/scholzri/rainy75-zmk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

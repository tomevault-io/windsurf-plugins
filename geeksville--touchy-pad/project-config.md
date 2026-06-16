---
trigger: always_on
description: Open-source multitouch USB touchpad / button matrix with a built-in
---

# touchy-pad — AI Agent Guide

Open-source multitouch USB touchpad / button matrix with a built-in
customisable LCD (ESP32-S3 and ESP32-P4; boards: jc4827w543, waveshare_s3_lcd_7b,
elecrow_s3_lcd_7, elecrow_s3_lcd_7_adv, elecrow_p4_lcd_7, squixl).
The host-side companion is a Python package (`touchy-pad`) that ships a
CLI (`touchy`), a high-level API, a Tkinter/PySide6 device simulator, and
a StreamDeck-compatibility shim (`TouchyDeck`).

`CLAUDE.md` is a symlink to this file — keep them in sync via the symlink.

## Repo layout
| Path | Purpose |
|------|---------|
| `firmware/main/` | ESP-IDF C++ firmware (CMake, **not** PlatformIO) |
| `firmware/main/main.cpp` | Entry point — keep thin; subsystems live in their own `.cpp/.h` |
| `firmware/boards/<board>/` | Per-board pinout / display / touch drivers |
| `proto/` | Shared protobuf schemas (`touchy.proto`, `widgets.proto`, `preferences.proto`) + nanopb `.options` files |
| `app/src/touchy_pad/` | Python package — `cli.py`, `client.py`, `transport.py`, `api/`, `sim/`, `touchydeck/`, `_proto/` |
| `app/tests/` | pytest suite (host-side only; firmware has no unit tests) |
| `tools/StreamController/` | git submodule, branch `pr-touchypad`, with `touchy_bootstrap.py` shim |
| `tools/streamdeck-probe/` | Stage 50.1 reverse-engineering tool |
| `docs/design.md` | **Authoritative stage history — read before starting new work** |
| `docs/host-api.md` | USB endpoint + protocol spec |
| `Justfile` | All build/test/run tasks — prefer `just <recipe>` over raw commands |
| `VERSION` | Single-source version (read by Python + CMake) |

## Implementation status
All stages 0–24.4, 50.2, 51, 64.1, 64.3, 64.4, 65, 65.1, 67, 68, 72, 81, 82, 83, 84, 85, and 86 are **done**. Latest active wire-format:
`Screen.Version.CURRENT == 5`, `SysBoardInfoResponse.ProtocolVersion.CURRENT == 9`,
`PreferencesFile.Version.CURRENT == 4`.
Highlights worth remembering:

- **Boards span two chips (Stage 65).** ESP32-S3 boards (`jc4827w543`,
  `waveshare_s3_lcd_7b`) have native USB; the classic-ESP32
  `esp32_2432s028rv3` ("CYD2USB") does not. Each board declares its chip
  in a one-line `firmware/boards/<board>/target` file; USB capability is
  keyed off `CONFIG_SOC_USB_OTG_SUPPORTED` (not a custom flag). `just
  firmware-reconfigure [board]` reads `target` and runs `idf.py
  -DBOARD=<board> set-target <chip>` — the `-DBOARD` is required, and
  `rm -f firmware/sdkconfig firmware/sdkconfig.<board>` before switching
  chips. `firmware/main/platform.{h,cpp}` exposes `platform_get()` →
  `{is_multitouch, has_usb}`, surfaced over proto as
  `SysBoardInfoResponse.is_multitouch` / `has_usb`.

- **The "CYD" family shares one C++ implementation (Stage 65.1).** All
  classic-ESP32 CYD boards (`esp32_2432s028rv3` 2.8" ST7789,
  `esp32_2432s024` 2.4" ILI9341, more coming) compile the *same* sources in
  `firmware/boards/cyd_common/` (`board.cpp`, `display.cpp`, `touch.cpp`).
  Each board dir keeps only its `board_pins.h` + a tiny
  `board/CMakeLists.txt` (which lists `../../cyd_common/*.cpp` and sets
  `PRIV_INCLUDE_DIRS "."` so the board's own `board_pins.h` wins) +
  `idf_component.yml`. No symlinks. The panel driver is chosen at compile
  time from `board_pins.h`: `cyd_common/display.cpp` keys off
  `BOARD_LCD_CONTROLLER_ILI9341` (pulls the `espressif/esp_lcd_ili9341`
  managed component, `esp_lcd_new_panel_ili9341`) vs the default
  `BOARD_LCD_CONTROLLER_ST7789` (in-tree `esp_lcd_new_panel_st7789`); bring-up
  is otherwise identical. ILI9341 typically wants `BOARD_LCD_INVERT_COLOR=0`
  where ST7789 wants `1`.

- USB device is a composite class: CDC-ACM + HID (mouse + keyboard via
  report IDs 1/2) + vendor-class bulk pair (command/response) + interrupt-IN
  mailbox endpoint (0x85) that just signals "events available".
- Host ↔ device wire protocol = self-synchronising frames
  `MAGIC(0xA5 0x5A) | LEN(u16 LE) | payload | CRC8` (Stage 64.3) over the
  bulk pair. Identical framing on every transport (USB, simulator TCP,
  serial). One decoder per side: `firmware/main/host_api.cpp` (the
  `HostApiLink` abstraction), `app/src/touchy_pad/transport.py`
  (`_StreamFramedTransport` / `_FrameDecoder`), and
  `rust/touchy-pad/src/transport.rs` (`FrameDecoder`). The serial
  transport (`transport_serial.py`; Rust `transport_serial.rs` behind the
  `serial` feature) always runs at 115200 baud and carries only protocol
  frames — device logs ride the Stage 64.1 `LogRecord` tunnel, never raw
  text on the protocol port. Firmware serial path is gated on
  `CONFIG_TOUCHY_PROTO_OVER_SERIAL` (default n; `y` for the CYD board,
  which has a `UartLink` on `UART_NUM_0` gated `#if
  CONFIG_TOUCHY_PROTO_OVER_SERIAL && !CONFIG_SOC_USB_OTG_SUPPORTED`).
  `firmware/main/CMakeLists.txt` REQUIRES `esp_driver_uart` for this
  (IDF v6 split out `driver/uart.h`). Gotcha: in `sdkconfig.defaults`,
  `# CONFIG_X is not set` is **not** a comment — it's the `X=n` directive
  and will silently override an earlier `CONFIG_X=y`.
- nanopb uses `FT_POINTER` (heap) for `repeated` widget/action/step
  fields and the `FileWrite` payload. RAII via `PbMessage<T>` in
  `firmware/main/protobuf.h`.
- Filesystem paths are drive-prefixed: `F:host/...` = LittleFS (persistent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geeksville/touchy-pad](https://github.com/geeksville/touchy-pad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

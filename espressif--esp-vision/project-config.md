---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.

ESP-VISION is a MicroPython vision runtime for ESP32-P4 / ESP32-S3 boards. It builds a custom firmware: the upstream MicroPython esp32 port plus an overlay, ESP-VISION C modules (`sensor`, `image`, `display`, `espdl`, `tflite`, the P4-only `h264` and `rtsp`, plus the `image.ImageIO` stream type), a self-written platform layer (camera, preview, display, storage, USB, H.264), and OpenMV's MIT `imlib` as a vision component. The VSCode host tool lives in a separate repo.

## Build Commands

The repository-root `idf.py --board <BOARD> ...` entry point is provided through `idf_ext.py`. It builds the generated MicroPython copy under `build/micropython/idf<ESP_IDF_VERSION>/micropython/ports/esp32`; never create a standalone IDF app at the repo root.

- Build: `idf.py --board ESP32_P4X_EYE build`
- Build + flash: `idf.py --board ESP32_P4X_EYE -p /dev/ttyACM0 build flash`
- Serial monitor: `idf.py --board ESP32_P4X_EYE -p /dev/ttyACM0 monitor`
- Menuconfig: `idf.py --board ESP32_P4X_EYE menuconfig`
- Size: `idf.py --board ESP32_P4X_EYE size`
- Erase flash: `idf.py --board ESP32_P4X_EYE erase-flash`
- Clean board build output: `idf.py --board ESP32_P4X_EYE clean`
- Wipe board build output: `idf.py --board ESP32_P4X_EYE fullclean`
- Requires ESP-IDF release/v5.5, release/v6.0, or master with `idf.py` on `PATH` (source the IDF `export.sh` if it isn't).

Notes:
- `BOARD` must exist as `boards/<BOARD>/` with a `boards/<BOARD>/port/` subdirectory (the MicroPython-port files projected onto the esp32 port at build time). Boards: `ESP32_P4X_EYE`, `ESP32_P4X_FUNCTION_EV_BOARD`, `ESP32_P4X_VISION`, `ESP32_S3_EYE`, `ESP32_S31_KORVO`, `AtomS3R-M12` (esp32s3); `TEMPLATE` is for new-board bring-up.
- `ESP32_S31_KORVO` requires ESP-IDF release/v6.1; source a release/v6.1 environment before building it.
- After changing the build system, board config, platform drivers, or imlib options, verify the `ESP32_P4X_EYE` build first; other boards only when the task touches them.
- Firmware build/flash/monitor/config commands first prepare the MicroPython tree: `idf_ext.py` asserts `lib/micropython` is at the pinned commit (`v1.28.0`, `e0e9fbb17ed6fd06bb76e266ae554784c9c80804`), recreates a clean MicroPython copy under `build/micropython/`, applies `overlay/micropython/` to that copy, then projects each `boards/<BOARD>/port/` onto the copy's `ports/esp32/boards/<BOARD>/`. This build-copy strategy does not dirty `lib/micropython`.

## Lint & Checks

There is no test suite; CI runs `pre-commit` and a firmware build matrix.

- C/C++ formatting (astyle 3.4.7): `pre-commit run --all-files astyle_py`
- Copyright/SPDX headers: `pre-commit run --all-files check-copyright`
- Stub syntax (mirrors `python_stubs` CI job): `python -m py_compile stubs/*.pyi`
- `components/imlib/`, `lib/micropython/`, `lib/ulab/`, `overlay/micropython/`, and OpenMV-sourced `modules/py_{helper,image,imageio,assert}.*` are excluded from formatting/copyright rewriting — do not re-include them or let the hooks touch upstream headers.

## Project Structure

Layered by whether code touches MicroPython (`mp_obj_t` / `py/*.h`):

- `components/imlib/`: pure-C vision algorithms, IDF component, maintained as MIT. `upstream/` = OpenMV `lib/imlib` MIT files (keep close to upstream, record any adaptation); `include/` = contract headers + CMSIS shims (no `.c`); `compat/` = ESP-VISION contract impls (`array.c`, `fb_alloc.c`, `umm_malloc.c`).
- `components/zxing/`: thin IDF component wrapping the `lib/zxing-cpp` submodule for the opt-in 1D barcode backend (`esp_vision_zxing.cpp` + `include/`/`compat/`). Only built into the `idf::zxing` target when a board sets `ESP_VISION_ENABLE_BARCODE`.
- `platform/`: self-written ESP32 glue, may couple with MicroPython. `ev_channel.c`, `ev_mux.c`, `ev_control_transport.c`, and `ev_stdio.c` implement the EV-MUX transport; `preview.c` sends binary JPEG `preview.frame` frames when `img.flush()` calls `esp_vision_preview_flush`. `display.c` is the generic display layer (the board provides panel/backlight via `esp_vision_board_display_*` hooks), `sdcard.c` mounts at `/sdcard` (the board provides `esp_vision_board_sdcard_*` weak hooks), `usb_msc.c` exposes the on-flash FAT data partition `ffat`/`vfs` over TinyUSB MSC, `jpeg.c` selects hardware JPEG or the `esp_new_jpeg` software fallback, `h264.c` is the P4-only hardware H.264 encoder over `esp_h264`, and `debug.c` routes debug output. `main.c` replaces upstream MP `main.c` and owns startup init + the soft-reset loop. `camera.c` here is only weak-symbol placeholders (`ESP_ERR_NOT_SUPPORTED`) plus the shared framesize table; the real backend lives per-board in `boards/<BOARD>/camera.c` (P4X uses `esp_video`/V4L2 + PPA, S3 uses `esp32-camera` — no esp_video/PPA).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [espressif/esp-vision](https://github.com/espressif/esp-vision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->

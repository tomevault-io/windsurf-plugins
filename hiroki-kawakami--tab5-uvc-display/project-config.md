---
trigger: always_on
description: ESP32-P4 (M5Stack Tab5) project that pulls 1280×720 MJPEG from a USB UVC camera,
---

# Tab5-UVC-Display — Claude notes

ESP32-P4 (M5Stack Tab5) project that pulls 1280×720 MJPEG from a USB UVC camera,
hardware-decodes it, and displays it rotated 90° on a 720×1280 MIPI-DSI panel.

## Build / flash / monitor

The dev environment lives in a Nix flake; **always use `nix develop`** instead
of sourcing `export.sh` directly:

```sh
nix develop --command idf.py -C esp32p4 build
nix develop --command idf.py -C esp32p4 flash monitor   # needs a TTY
```

When `idf.py monitor` can't attach (no TTY in non-interactive shell), drive
the serial directly with PySerial:

```sh
nix develop --command python3 -u -c "
import subprocess, serial, time, sys
subprocess.run(['python', '-m', 'esptool', '--chip', 'esp32p4', '-p',
    '/dev/cu.usbmodem114201', '-b', '460800', '--before=default_reset',
    '--after=hard_reset', 'write_flash', '--flash_mode', 'dio',
    '--flash_freq', '80m', '--flash_size', '16MB',
    '0x10000', 'esp32p4/build/tab5-uvc-display.bin'])
ser = serial.Serial('/dev/cu.usbmodem114201', 115200, timeout=0.2)
end = time.time() + 15
while time.time() < end:
    data = ser.read(4096)
    if data: sys.stdout.write(data.decode('utf-8', errors='replace')); sys.stdout.flush()
"
```

Note: the second `/dev/cu.usbmodem*` enumerator is the JTAG/console port we
flash on; the first one is busy if the user already has a monitor open.
**ESP32-P4 only prints logs once after reset** — capture during the boot
sequence, don't expect output later.

ESP-IDF v5.4.3 lives at `/nix/store/1jf3iqwyp77i8y54cgn6qxbrwl3wx5mz-esp-idf-v5.4.3/`.

## Layout

- `app/` — `PreviewScreen` (UVC frame → pipeline → display), `uvc_display.cpp`.
- `components/{lvgl++,screen_manager}/` — shared UI helpers.
- `idf-components/main/` — IDF entry (`main.cpp`), `platform_port_*` adapters
  for JPEG/PPA, USB host wrappers. **No explicit `REQUIRES`** — the absence
  is intentional so that the implicit "all-components-available" mode stays
  on; adding `REQUIRES`/`PRIV_REQUIRES` here breaks transitive header lookup
  (e.g. `bsp_tab5.h`, `esp_timer.h`).
- `idf-components/m5tab5-bsp/` — vendored BSP for the panel + touch +
  audio. `inc/audio_eq.h` + `src/audio_eq.c` is the cascaded-biquad EQ /
  software fader / mono-mix DSP block that sits inside
  `bsp_tab5_audio_write` (see the audio section below).
- `idf-components/jpeg_decode_enhanced/` — reusable strip-pipelined JPEG
  decode (+ optional PPA) component, all C. Full documentation (usage,
  config reference, tuning, 2D-DMA internals) lives in its `README.md`.
  Two layers:
  - `jpeg_decode_enhanced.h` — Layer 1: strip decoder
    (`jpeg_enh_strip_decoder_*`) + whole-frame convenience decode
    (`jpeg_enh_decoder_process`). PPA-free; full-range YUV→RGB capable.
  - `jpeg_ppa_pipeline.h` — Layer 2: `jpeg_ppa_pipeline_*`, drives Layer 1
    strips through PPA SRM with a per-frame transform (rotation / scale /
    mirror / crop / output offset).
  Bypasses IDF's `jpeg_decoder_process()` by reaching into ESP-IDF private
  headers; do **not** override `esp_driver_jpeg` (the user explicitly rejected
  that approach). The component's CMakeLists adds private include paths via
  `target_include_directories(... PRIVATE $ENV{IDF_PATH}/components/esp_driver_jpeg{,/private})`,
  and `jpeg_decode_enhanced.c` has an `ESP_IDF_VERSION` guard (validated on
  v5.4.x only) because it touches `jpeg_private.h` struct layout.

`esp32p4/CMakeLists.txt` wires both `components/` and `idf-components/` via
`EXTRA_COMPONENT_DIRS`. The `main` component's CMake also globs `app/` and
`components/*/` into the main source list.

## jpeg_decode_enhanced — design summary

Why it exists: JPEG-codec alone can do 60fps@1280×720; with the stock
`jpeg_decoder_process` → PSRAM → PPA SRM → PSRAM-FB chain, PSRAM bandwidth
caps throughput at ~20fps. Decoding into a ring of **internal SRAM** strip
buffers and feeding each strip through PPA SRM in parallel removes the
PSRAM-read leg and brings the system back to camera-saturating 30fps.
(`strip_alloc_caps = MALLOC_CAP_SPIRAM` keeps the ring in PSRAM instead —
slower, but useful when the goal is just a small intermediate buffer; the
component handles the cache purge at alloc time and CPU consumers must call
`jpeg_enh_strip_decoder_sync_strip_for_cpu` before reading a strip.)

Frame geometry is re-derived from the JPEG header every frame: any size up
to `max_pic_w/max_pic_h` decodes without reconfiguration, `strip_h_hint` is
rounded to the frame's MCU height, the final strip may be shorter, and
non-MCU-aligned image heights are decoded padded then cropped by the PPA
stage (strip events carry `rows` = valid vs `padded_rows`). The only hard
constraint left is hardware: strip boundaries sit on MCU-row boundaries
because the 2D-DMA RX reorder works in JPEG-sampling-sized macro blocks and
one MCU row cannot span two descriptors.

`yuv_full_range` (Layer 1 cfg / pipeline cfg) switches the decode CSC from
IDF's stock limited-range matrix to the JFIF full-range one (BT601 and BT709
tables both provided). Default **false** = IDF-compatible limited range;
the Tab5 app sets **true** — forgetting it washes out blacks.

Pipeline shape:

```
JPEG codec ──TX from PSRAM JPEG stream──┐
                                        ▼

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hiroki-Kawakami/Tab5-UVC-Display](https://github.com/Hiroki-Kawakami/Tab5-UVC-Display) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

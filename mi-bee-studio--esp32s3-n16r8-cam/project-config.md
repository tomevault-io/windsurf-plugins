---
trigger: always_on
description: > Firmware project for an ESP32-S3-N16R8 module + OV3660 camera. **Production-ready** with MJPEG streaming, AI detection, RTSP, ONVIF, and responsive web UI.
---

# AGENTS.md — ESP32-S3-N16R8 CAM

> Firmware project for an ESP32-S3-N16R8 module + OV3660 camera. **Production-ready** with MJPEG streaming, AI detection, RTSP, ONVIF, and responsive web UI.

## AT command interface (family contract v1.0, 2026-09-04)

统一契约：`docs/at-command.md`（四仓 md5 一致，地位同 api-contract）。核心集：
`AT / AT+HELP / AT+GMR / AT+STATUS / AT+WIFI?|= / AT+IP? / AT+CAMRES?|= / AT+CAMQUAL?|= /
AT+REBOOT / AT+RESTORE`（+能力裁剪项）。红线：**任何读指令不回显密码**；CAMQUAL 边界
10-63（PIT-021）。本板串口 /dev/ttyUSB1（CH340）。CAMRES/CAMQUAL 走 camera_reinit 热重配（AI 全开时锁 VGA）；实现于 main/at_command.c（含 AI/LED/RTSPPASS 扩展）。

## Hardware target

| Item | Value | Notes |
|------|-------|-------|
| Module | ESP32-S3-WROOM-1 **N16R8** | N16 = 16 MB Quad Flash · R8 = 8 MB **Octal** PSRAM |
| SoC | ESP32-S3 (Xtensa LX7 dual-core @ 240 MHz) | USB-OTG + USB-Serial/JTAG |
| Camera | **OV3660** (3 MP, 1/5", max QXGA 2048×1536) | NOT the OV2640 from the reference repos — see below |
| USB | USB-Serial/JTAG (enumerates as `/dev/ttyACM0`) | |

### Why N16R8 changes the design vs the reference repos

- **16 MB Flash** (vs 8 MB on `seeed-esp32s3-cam`, 4 MB on `ai-thinker-esp32-cam`) → partition table can hold two **larger** OTA slots, bigger SPIFFS/NVS, or a factory image. Re-plan `partitions.csv` from scratch; do **not** copy the 8 MB layout.
- **8 MB Octal PSRAM** → same module family as the XIAO board, so the S3-Octal gotchas carry over (see *Octal PSRAM* below).
- **OV3660 ≠ OV2640**:
  - Higher max resolution (QXGA). Defaults copied from OV2640 (typically UXGA) will misallocate frame buffers.
  - esp32-camera exposes it via the same `esp_camera_*` API; `PIXFORMAT_JPEG` still works, but JPEG engine quality/rate differs.
  - Sensor ID is `0x77` (OV2640 is `0x26`/`0x42`) — useful for `esp_camera_sensor_get()` sanity checks.

## Toolchain

| Tool | Version | Path / Notes |
|------|---------|--------------|
| ESP-IDF | **v6.0.1 (pinned)** | `~/.espressif/v6.0.1/esp-idf/` |
| Component: `espressif/esp32-camera` | `^2.1.6` | Proven across both reference repos, supports OV3660 |
| Target | `esp32s3` | Set once per build dir |

Activation (every new shell):
```bash
source ~/.espressif/v6.0.1/esp-idf/export.sh
```

## Reference repos (carry conventions forward, do NOT copy pin tables)

| Repo | What to steal | What NOT to copy |
|------|---------------|------------------|
| https://github.com/Mi-Bee-Studio/seeed-esp32s3-cam | S3 + Octal PSRAM sdkconfig patterns, `main/` flat module layout, dual-OTA partitioning, SPIFFS web UI embedding, build/flash/release workflow | XIAO ESP32-S3 Sense **camera pin map** (different board) |
| https://github.com/Mi-Bee-Studio/ai-thinker-esp32-cam | Simpler motion-detect/MJPEG core, ESP-IDF v6.0.1 CI badge convention | Everything ESP32-specific (plain ESP32 has no Octal PSRAM, DMA differs, IRAM pressure tuning is ESP32-only) |

The `seeed-esp32s3-cam` repo has a detailed `AGENTS.md` worth reading for S3 patterns; this file is its sibling, scoped to N16R8 + OV3660.

## Shipped Features

The firmware is production-ready with the following modules and features:

### Core Modules (15 modules)

| Module | Files | Purpose |
|--------|-------|---------|
| main.c | main.c | App entry, boot sequence orchestrator |
| config_manager | config_manager.c/h | NVS-backed config, 16 keys, TYPE_U8/TYPE_I8 |
| camera_driver | camera_driver.c/h | OV3660 init, sensor settings, coordinated reinit |
| frame_broadcaster | frame_broadcaster.c/h | Frame grab task on Core 1, publisher-subscriber pattern |
| mjpeg_streamer | mjpeg_streamer.c/h | HTTP MJPEG streaming via chunked multipart |
| ai_pipeline | ai_pipeline.cpp/h | Face + motion + QR detection, 640×480 buffers, ESP-DL |
| web_server | web_server.c/h | REST API (11 endpoints), SPIFFS static files |
| web_ui | index.html, style.css, app.js, i18n.js | Browser UI, zh/en i18n, light/dark theme |
| wifi_manager | wifi_manager.c/h | WiFi STA mode connection |
| flash_led | flash_led.c/h | GPIO flash LED control |
| at_command | at_command.c/h | Serial AT command interface |
| rtsp_server | rtsp_server.cpp/h | RTSP server with MJPEG-only streaming |
| onvif_service | onvif_service.c/h | ONVIF SOAP service |
| onvif_discovery | onvif_discovery.c/h | ONVIF WS-Discovery protocol |
| status_led | status_led.c/h | GPIO status LED |

### REST API Endpoints

> **2026-09-02 契约 v1.0 统一化**（权威规范：`docs/api-contract.md`，下表已过时）：
> 新增核心端点 `GET /api/capture`、`GET /api/scan`、`POST /api/reset`、`POST /api/reboot`、
> `POST /api/time`、`GET /api/auth`、`GET /metrics`、`GET /api/led`；
> `POST /api/camera` framesize 合法域收窄为 0-15（与广播的 `supported_resolutions` 一致）；
> capabilities 增加 `api_version`/`wifi_scan`；status 字段对齐契约
> （`camera_resolution`→`resolution`、`mjpeg_clients`→`stream_clients`，新增 `camera`/`device_name`/
> `firmware_version`/`wifi_state`/`min_heap`/`stream_clients_max`）；config 新增 `device_name` 键。
> **RTSP 鉴权已启用**：vendored `components/espp__rtsp`（自 seeed 复制，含 digest 补丁）+
> 直接依赖 `espp/base_component|socket|task`（idf_component.yml 已改，勿再加 espp/rtsp）。
> ONVIF GetSnapshotUri 已指向 `:80/api/capture`。
>
> **契约 v1.1（2026-09-02）**：移植 seeed `ota_updater`（`/api/ota`、`/api/ota/info`、

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mi-Bee-Studio/esp32s3-n16r8-cam](https://github.com/Mi-Bee-Studio/esp32s3-n16r8-cam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

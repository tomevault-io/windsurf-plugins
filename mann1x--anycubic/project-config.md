---
trigger: always_on
description: Development tools and utilities for Anycubic RV1106-based 3D printers (Kobra 2 Pro, Kobra 3, Kobra S1, etc.).
---

# Anycubic Tools - Development Guide

Development tools and utilities for Anycubic RV1106-based 3D printers (Kobra 2 Pro, Kobra 3, Kobra S1, etc.).

## Repository Structure

```
anycubic/
├── rkmpi-encoder/      # Hardware H.264/JPEG encoder (C)
├── h264-streamer/      # HTTP streaming server (C)
├── fb-status/          # Framebuffer status display (C)
├── scripts/            # Test and utility scripts
└── knowledge/          # Protocol documentation
```

## Components

### rkmpi-encoder
Native USB camera capture with RV1106 hardware H.264 encoding.
- V4L2 MJPEG/YUYV capture from USB cameras
- TurboJPEG software decode OR hardware JPEG encoding
- RKMPI VENC hardware H.264 encoding
- Built-in HTTP, MQTT, and RPC servers
- **Control server** (port 8081) with web UI, REST API, config persistence
- **Multi-camera management** - camera detection, fork/exec secondary instances
- **Display capture** with full hardware acceleration (RGA + VENC)
- **Timelapse recording** with hardware VENC encoding (no ffmpeg dependency)
- **Moonraker WebSocket client** - timelapse layer/hyperlapse triggers
- **MQTT keepalive** - automatic PINGREQ to prevent broker disconnections
- See: `rkmpi-encoder/claude.md`

### h264-streamer
Deployment package for Rinkhals custom firmware (pure C, no Python).
- Shell scripts (app.sh, h264_monitor.sh) + rkmpi_enc binary
- HTML templates for web UI (control.html, timelapse.html, index.html)
- All streaming, control, and timelapse logic is in the rkmpi_enc binary
- See: `h264-streamer/claude.md`

### fb-status
Lightweight framebuffer text overlay utility.
- Direct /dev/fb0 rendering with TTF fonts
- Screen save/restore via ffmpeg
- Pipe mode for long-running scripts
- Auto screen orientation detection
- See: `fb-status/claude.md`

---

## Target Hardware: RV1106

### Specifications
| Component | Specification |
|-----------|---------------|
| CPU | Single-core ARM Cortex-A7 @ 1.2GHz |
| Architecture | ARMv7-A with NEON SIMD |
| RAM | 256MB DDR3 (100-150MB available) |
| Video Encoder | H.264/H.265 up to 2304x1296 @ 30fps |
| USB | USB 2.0 OTG (~35-40 MB/s practical) |

### Printer Models
| Code | Model ID | Printer |
|------|----------|---------|
| K2P | 20021 | Kobra 2 Pro |
| K3 | 20024 | Kobra 3 |
| KS1 | 20025 | Kobra S1 |
| K3M | 20026 | Kobra 3 Max |
| K3V2 | 20027 | Kobra 3 V2 |
| KS1M | 20029 | Kobra S1 Max |

---

## Build System

### Cross-Compilation Toolchain
```bash
# Toolchain location
/shared/dev/rv1106-toolchain

# Compiler prefix
arm-rockchip830-linux-uclibcgnueabihf-

# Compiler flags
-march=armv7-a -mfpu=neon -mfloat-abi=hard -fno-stack-protector -O2
```

### Build Commands
```bash
# Build rkmpi-encoder
cd rkmpi-encoder
make dynamic          # Dynamic linking (~2MB)
make static           # Static linking (~5MB+)
make install-h264     # Copy to h264-streamer

# Build fb-status
cd fb-status
make                  # Build binary (~70KB)
make deploy           # Deploy to printer
```

### Required Libraries (on printer)
Located in `/oem/usr/lib/`:
- librockit_full.so - Multimedia framework
- librockchip_mpp.so - Media processing
- librga.so - 2D graphics
- libdrm.so - Direct rendering
- libturbojpeg.so - JPEG codec

---

## Printer Connection

```bash
PRINTER_IP=192.168.178.43

# SSH access
sshpass -p 'rockchip' ssh root@$PRINTER_IP

# Deploy files
sshpass -p 'rockchip' scp ./binary root@$PRINTER_IP:/tmp/

# Run on printer
sshpass -p 'rockchip' ssh root@$PRINTER_IP '/tmp/binary'
```

---

## Deployment

### Deploy h264-streamer to Printer

```bash
PRINTER_IP=192.168.178.43
APP_DIR=/useremain/home/rinkhals/apps/29-h264-streamer

# 1. Build encoder
cd rkmpi-encoder
make dynamic

# 2. Stop h264-streamer
sshpass -p 'rockchip' ssh root@$PRINTER_IP "$APP_DIR/app.sh stop"

# 3. Deploy binary
sshpass -p 'rockchip' scp rkmpi_enc root@$PRINTER_IP:$APP_DIR/rkmpi_enc

# 4. Start h264-streamer
sshpass -p 'rockchip' ssh root@$PRINTER_IP "$APP_DIR/app.sh start"

# 5. Check status and logs
sshpass -p 'rockchip' ssh root@$PRINTER_IP "$APP_DIR/app.sh status"
sshpass -p 'rockchip' ssh root@$PRINTER_IP "tail -20 /tmp/rinkhals/app-h264-streamer.log"
```

### Important Paths on Printer

| Path | Description |
|------|-------------|
| `/useremain/home/rinkhals/apps/29-h264-streamer/` | h264-streamer app directory |
| `/useremain/home/rinkhals/apps/29-h264-streamer.config` | Persistent config file (JSON) |
| `/tmp/h264_cmd` | Command file for CAM#1 (control server → encoder) |
| `/tmp/h264_ctrl` | Control/stats file for CAM#1 (encoder → control server) |
| `/tmp/h264_cmd_N` | Command file for CAM#N (N=2,3,4) |
| `/tmp/h264_ctrl_N` | Control/stats file for CAM#N |
| `/tmp/rinkhals/app-h264-streamer.log` | Application log file |

### Control File Format

Each camera has separate command and control files. For CAM#1, they are `/tmp/h264_cmd` and `/tmp/h264_ctrl`. For CAM#2-4, they are `/tmp/h264_cmd_N` and `/tmp/h264_ctrl_N`.

**Command file (written by control server for secondary cameras):**
- `h264=0|1` - Enable/disable H.264 encoding
- `display_enabled=0|1` - Enable/disable display capture
- `display_fps=N` - Display capture frame rate
- `mjpeg_fps=N` - Target MJPEG frame rate
- `cam_brightness=N` - Camera brightness (0-255)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mann1x/anycubic](https://github.com/mann1x/anycubic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

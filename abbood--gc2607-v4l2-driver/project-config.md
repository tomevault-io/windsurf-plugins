---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Current Status Summary

**✅ CAMERA IS WORKING AND CAPTURING IMAGES!**

**What's Complete (Phases 1-6):**
- ✅ Driver fully functional with V4L2 integration
- ✅ IPU6 bridge integration (modified ipu_bridge.ko installed)
- ✅ Successfully capturing 1920x1080@30fps RAW Bayer images
- ✅ Media pipeline: gc2607 → IPU6 CSI2 0 → /dev/video0
- ✅ Image viewer scripts with brightness adjustment

**Phase 7: Exposure, Gain & White Balance ✅ COMPLETE:**
- ✅ Exposure control (V4L2_CID_EXPOSURE) - range 4-2002
- ✅ Gain control (V4L2_CID_ANALOGUE_GAIN) - LUT index 0-16
- ✅ Gray world white balance (R=1.034, G=1.000, B=1.246)
- ✅ Optimal settings for indoor lighting: exposure=2002, gain=16
- ✅ Real-time white balance in GStreamer pipeline using frei0r

**Quick Capture Test:**
```bash
sudo modprobe videodev v4l2-async ipu_bridge intel-ipu6 intel-ipu6-isys
sudo insmod gc2607.ko
v4l2-ctl -d /dev/video0 --set-fmt-video=width=1920,height=1080,pixelformat=BA10
media-ctl -d /dev/media0 -l '"Intel IPU6 CSI2 0":1 -> "Intel IPU6 ISYS Capture 0":0[1]'
v4l2-ctl -d /dev/video0 --stream-mmap --stream-count=1 --stream-to=test.raw
./view_raw_wb.py test.raw 5.0 && feh test.png
```

**Note:** Use `view_raw_wb.py` (with white balance) for natural colors, or `view_raw_bright.py` (without WB) for quick testing.

## Project Overview

This project successfully ported the GalaxyCore GC2607 camera sensor driver from the Ingenic T41 platform (MIPS embedded) to the Linux V4L2 subsystem for Intel IPU6 on x86_64.

**Target Hardware:**
- Laptop: Huawei MateBook Pro VGHH-XX
- Sensor: GC2607 (1920x1080@30fps, MIPI CSI-2, RAW10)
- Platform: Intel IPU6
- PMIC: INT3472:01 discrete (intel_skl_int3472_discrete driver)
- I2C Bus: /dev/i2c-5
- I2C Address: 0x37
- Chip ID: 0x2607 (registers 0x03f0=0x26, 0x03f1=0x07) ✅ VERIFIED

**ACPI Matching:**
- Device name: GCTI2607:00
- ACPI path: `\_SB_.PC00.LNK0`
- Modalias: `acpi:GCTI2607:GCTI2607:`
- Driver uses ACPI match table with "GCTI2607" HID

**INT3472 PMIC Resources (INT3472:01):**
- Regulator: `INT3472:01-avdd` (used by sensor)
- Privacy LED: `GCTI2607_00::privacy_led`
- Reset GPIO: Provided via ACPI
- Clock: 19.2 MHz from platform
- Status: Enabled and bound to `int3472-discrete`

## Architecture

### Reference Driver (reference/gc2607.c)
The original Ingenic T41 driver uses platform-specific APIs:
- `tx-isp-common.h`, `sensor-common.h`: T41 ISP framework
- `private_i2c_transfer()`, `private_gpio_request()`: T41-specific wrappers
- Platform device registration with `tx_isp_subdev` abstraction

### Implemented V4L2 Driver (gc2607.c)
The new driver implements:
1. ✅ Standard Linux V4L2 subdev APIs
2. ✅ I2C client driver with ACPI match table
3. ✅ V4L2 subdev ops (video, pad)
4. ✅ GPIO/regulator APIs via INT3472 PMIC
5. ✅ Async subdev registration for IPU6
6. ✅ V4L2 controls (link frequency, pixel rate)

### Key Hardware Configuration
Confirmed from hardware testing:
- MIPI: 2 lanes, 672 Mbps/lane (link_freq=336MHz)
- Pixel format: SGRBG10 (Bayer GRBG 10-bit)
- Resolution: 1920x1080@30fps
- Frame timing: HTS=2048, VTS=1335
- Register addressing: 16-bit addresses, 8-bit values
- Initialization: 122 register writes

## Development Workflow

### Building the Driver
```bash
# Out-of-tree build against running kernel
make

# Clean build artifacts
make clean
```

### Testing the Driver
```bash
# Quick test (recommended)
sudo ./test_phase4.sh

# Load module manually
sudo insmod gc2607.ko

# Check probe status
dmesg | grep gc2607

# Check V4L2 registration
v4l2-ctl --list-subdevs

# Unload module
sudo rmmod gc2607
```

### Camera Integration Testing
```bash
# Test IPU6 integration
sudo ./test_camera_streaming.sh

# Check media controller topology
media-ctl -d /dev/media0 --print-topology

# Investigate ipu_bridge
sudo ./investigate_ipu_bridge.sh
```

## Implementation Status

### Phase 1: Skeleton Driver ✅ COMPLETE
**Status:** Fully working
- I2C client registration with ACPI matching
- Basic probe/remove with logging
- Module metadata and build system

**Test:** Module loads and binds to ACPI device

### Phase 2: Power Management ✅ COMPLETE
**Status:** Fully working
- INT3472 PMIC integration (GPIOs, regulators, clocks)
- Proper reset sequence: HIGH (20ms) → LOW (20ms) → HIGH (10ms)
- Sensor detection confirmed (chip ID 0x2607)
- Power on/off sequences working

**Test:** `sudo ./QUICK_TEST.sh` shows chip ID 0x2607

**Key Achievement:** Fixed critical reset sequence bug where sensor was left in reset state

### Phase 3: Register Initialization ✅ COMPLETE
**Status:** Fully working
- 122-register initialization sequence from reference driver
- Register write functions implemented
- Integrated into s_stream() for streaming start
- Register array with proper handling of delays

**Test:** `sudo ./test_phase3.sh` confirms all registers ready

**Files:** Register table `gc2607_1080p_30fps_regs[]` in gc2607.c

### Phase 4: V4L2 Integration ✅ COMPLETE
**Status:** Fully working
- V4L2 pad operations (enum_mbus_code, enum_frame_size, get_fmt, set_fmt)
- V4L2 controls (link_freq=336MHz, pixel_rate=134.4MHz)
- Async subdev registration
- Format: SGRBG10 1920x1080@30fps

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abbood/gc2607-v4l2-driver](https://github.com/abbood/gc2607-v4l2-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

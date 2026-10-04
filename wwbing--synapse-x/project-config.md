---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Synapse-X is an ultra-low-latency dual-machine AI visual inference pipeline with real-time aim assist. A **Host** (gaming PC) captures the screen at 170Hz, compresses frames with LZ4, and sends them over a dedicated Ethernet cable to a **Client** (inference PC) running TensorRT YOLO. The Client returns detection coordinates, and the Host drives mouse movement via `ddll64.dll`.

All comments and documentation are in Chinese. Code identifiers are in English.

## Build Commands

Host and Client are built **independently** on separate physical machines. There is no top-level build.

### Host (gaming machine)
```powershell
cd host
cmake --preset windows-x64
cmake --build build_x64 --config RelWithDebInfo
```

### Client (inference machine, requires CUDA + TensorRT)
```powershell
cd client
cmake --preset windows-x64
cmake --build build_x64 --config RelWithDebInfo
```

### Test program (Host only)
The Host build also produces `SynapseX_Host_TestBmp.exe` — a standalone BMP capture + LZ4 compression test. Built as part of the Host build above.

### Client inference test
```powershell
cd client\test
cmake --preset windows-x64
cmake --build build_x64 --config RelWithDebInfo
```
Produces `test_infer.exe` which runs TensorRT inference against test images (`test/image/*.jpg`) and compares against expected results (`test/result/*_detections.txt`). Links WIC and COM for image loading.

## Architecture

```
Host (gaming PC)                          Client (inference PC)
┌──────────────────────┐                  ┌─────────────────────────┐
│ DXGI → LZ4 → UDP :8888 ────dedicated───→ Reassembly → LZ4 decode │
│                      │     Ethernet     │                         │
│ MouseCtrl ← UDP :8889 ←───────────────── TensorRT YOLO inference  │
│                      │                  │                         │
│ HttpTuner :9999      │                  │                         │
└──────────────────────┘                  └─────────────────────────┘
```

### Shared Protocol (`shared/include/`)

Header-only library used by both sides via relative path `../shared/include/`.

- **`PacketHeader.h`** — Host→Client frame metadata (24 bytes): magic `0x5358`, frameId, chunking info, ROI dimensions, modelId. Max payload 1400 bytes per UDP datagram.
- **`ReplyPacket.h`** — Client→Host detection results: `ReplyHeader` (16 bytes, magic `0x5359`) + up to 50 `DetectionRaw` structs (24 bytes each: x1,y1,x2,y2,confidence,classId).
- **`Log.h`** — spdlog-based logging setup with console + rotating file sinks. Defines `SX_LOG_*` macros (`SX_LOG_TRACE` through `SX_LOG_CRITICAL`). Call `SynapseX::Log::Initialize(appName)` at startup.

### Host Modules (`host/`)

| Module | Header | Role |
|--------|--------|------|
| DxgiCapturer | `include/DxgiCapturer.h` | DXGI Desktop Duplication, GPU-side ROI crop, 170Hz fixed-rate capture, thread pinning to P-Core |
| Lz4Compressor | `include/Lz4Compressor.h` | LZ4 block compression with `LZ4_compress_fast(accel=5)` |
| UdpSender | `include/UdpSender.h` | Chunks compressed frame into ≤1400B packets with PacketHeader, non-blocking 4MB buffer |
| UdpReplyReceiver | `include/UdpReplyReceiver.h` | Listens on UDP :8889, deserializes ReplyHeader + DetectionRaw[], maps coordinates to screen space |
| MouseController | `include/MouseController.h` | PD controller with dynamic Kp, Kd damping, sub-pixel accumulator, 2-frame delay compensation, loads `ddll64.dll` |
| HttpTuner | `include/HttpTuner.h` | Embedded HTTP server on :9999 serving `web/index.html` for real-time parameter tuning from phone/tablet |

### Client Modules (`client/`)

| Module | Header | Role |
|--------|--------|------|
| UdpReceiver | `include/UdpReceiver.h` | UDP recv on :8888, decompresses with LZ4 |
| ReassemblyBuffer | `include/ReassemblyBuffer.h` | Out-of-order packet reassembly by frameId |
| TrtInference | `include/TrtInference.h` | TensorRT YOLO inference, FP16, 416×416, loads `.engine` file |
| CudaPreprocess | `include/CudaPreprocess.h` | GPU-side image preprocessing via NVRTC |
| UdpReplySender | `include/UdpReplySender.h` | Sends detection results back to Host on :8889 |

## Key Architectural Details

### Host Main Loop (170 Hz fixed-rate, `host/src/main.cpp`)

Each tick (~5.882ms) executes these stages in order:

1. **Hotkey polling** — `GetAsyncKeyState` edge-triggered detection for PageUp (enable) / PageDown (disable) aim assist.
2. **DXGI capture** — `DxgiCapturer::CaptureFrame()` with `AcquireNextFrame(timeout=0)`. If no new frame, reuses cached compressed data.
3. **LZ4 compression** — Only when new frame arrives. `LZ4_compress_fast(accel=5)`. Result cached for subsequent re-sends.
4. **UDP send** — Always sends every tick (new frame or cached repeat). `frameId` increments monotonically. Chunks into 1400-byte packets via stack-allocated buffer — zero heap allocation on hot path.
5. **Reply receive** — Non-blocking drain of all queued datagrams on port 8889. Maps model-space coordinates to screen-space via ROI offset.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wwbing/Synapse-X](https://github.com/wwbing/Synapse-X) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->

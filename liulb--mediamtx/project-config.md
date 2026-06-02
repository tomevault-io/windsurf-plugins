---
trigger: always_on
description: This is a self-hosted telemedicine video streaming system that replaces Tencent Cloud TRTC service to achieve 90% cost savings (from ¥7,300/month to ¥700/month). The system enables **bidirectional video and audio communication** between a web-based doctor client and a WeChat mini-program patient client.
---

# MediaMTX Telemedicine Video System

## Project Overview

This is a self-hosted telemedicine video streaming system that replaces Tencent Cloud TRTC service to achieve 90% cost savings (from ¥7,300/month to ¥700/month). The system enables **bidirectional video and audio communication** between a web-based doctor client and a WeChat mini-program patient client.

**Key Achievement**: Bidirectional video streaming with 3-5 second latency using MediaMTX as the central media server, with FFmpeg-based audio transcoding for cross-platform compatibility.

## Architecture

```
┌─────────────────┐          ┌─────────────────┐          ┌──────────────────┐          ┌─────────────────┐
│   Doctor端      │          │   MediaMTX      │          │   FFmpeg         │          │   患者端        │
│   (Web)         │          │   Server        │          │   Transcoder     │          │  (小程序)       │
├─────────────────┤          ├─────────────────┤          ├──────────────────┤          ├─────────────────┤
│ Push: WebRTC    │─ WHIP ─>│ Port 8889       │          │ Input: RTSP      │          │ Push: RTMP      │
│ (H.264+Opus)    │          │ (Opus audio)    │─ RTSP ─>│ (Opus→AAC)       │─ RTMP ─>│ (H.264+AAC)     │
│                 │          │                 │          │ Output: RTMP     │          │                 │
│ Play: HLS       │<─ HLS ──│ Port 8888       │<─ RTMP ─│ (AAC audio)      │          │ Play: RTMP      │
│ (fMP4)          │          │ (AAC audio)     │          │                  │          │ (AAC audio)     │
└─────────────────┘          └─────────────────┘          └──────────────────┘          └─────────────────┘
                                      ↑                                                            │
                                      └────────────────── RTMP ────────────────────────────────────┘
```

### Technology Stack

- **MediaMTX**: Central streaming media server (bluenviron/mediamtx:latest)
  - Handles WebRTC (WHIP), RTMP, RTSP, HLS protocols
  - Real-time protocol conversion

- **FFmpeg Transcoder**: Audio codec conversion (jrottenberg/ffmpeg:6-alpine)
  - Converts Opus (WebRTC) → AAC (RTMP/iOS compatible)
  - Video passthrough (no re-encoding)

- **Doctor端 (web-doctor/)**:
  - React + Vite
  - WebRTC push via WHIP protocol (H.264 + Opus)
  - HLS playback with hls.js (supports fMP4 + AAC)
  - Forced H.264 encoding via SDP manipulation

- **患者端 (miniapp-patient/)**:
  - WeChat Mini-Program
  - RTMP push via live-pusher component (H.264 + AAC)
  - RTMP playback via live-player component (requires AAC audio)
  - Manual audio playback trigger for iOS

### Port Mapping

| Port | Protocol | Purpose |
|------|----------|---------|
| 1935 | RTMP | Mini-program RTMP push/pull, FFmpeg output |
| 8554 | RTSP | MediaMTX RTSP server, FFmpeg input |
| 8888 | HTTP | HLS playback endpoint |
| 8889 | HTTP | WebRTC/WHIP endpoint |
| 8189 | UDP | WebRTC ICE/media |
| 9997 | HTTP | MediaMTX API (requires auth) |
| 5173 | HTTP | Vite dev server (doctor端) |

## Common Commands

### Start All Services

```bash
cd /document/livekit-demo
docker-compose up -d

# View logs
docker logs -f mediamtx
docker logs -f transcoder
```

### Start Doctor端 (Web Client)

```bash
cd web-doctor
npm run dev  # Runs on http://localhost:5173
```

### Useful Docker Commands

```bash
docker-compose down               # Stop all services
docker-compose restart mediamtx   # Restart MediaMTX
docker-compose restart transcoder # Restart audio transcoder
docker logs mediamtx --tail 50    # View MediaMTX logs
docker logs transcoder --tail 50  # View FFmpeg logs
```

### Testing Endpoints

```bash
# Test HLS streams (in browser)
http://192.168.20.209:8888/doctorStream_aac/index.m3u8  # Doctor stream (AAC audio)
http://192.168.20.209:8888/patientStream/index.m3u8     # Patient stream

# Test RTMP streams (using FFplay)
ffplay rtmp://192.168.20.209:1935/doctorStream_aac
ffplay rtmp://192.168.20.209:1935/patientStream
```

## Key Files and Their Purposes

### Core Configuration

- **config/mediamtx.yml**: MediaMTX server configuration
  - Enables RTMP, RTSP, HLS, WebRTC services
  - HLS settings: `hlsVariant: mpegts`, 3 segments, 1s duration
  - WebRTC ICE configuration with STUN server
  - NAT IP configuration: `webrtcICEHostNAT1To1IPs: ['192.168.20.209']`

- **docker-compose.yml**: Container orchestration
  - MediaMTX container (ports, config mounting)
  - FFmpeg transcoder container (audio conversion)
  - Network mode: host (for inter-container communication)

- **transcode.sh**: FFmpeg transcoding script
  - Reads from RTSP: `rtsp://192.168.20.209:8554/doctorStream`
  - Outputs to RTMP: `rtmp://192.168.20.209:1935/doctorStream_aac`
  - Video: copy (no re-encoding)
  - Audio: Opus → AAC (128 kbps, 44.1 kHz)

### Doctor端 (Web)

- **web-doctor/src/App.jsx**: Main application component
  - Lines 13-84: HLS playback with hls.js (fMP4 + AAC support)
  - Lines 86-256: WebRTC push via WHIP protocol
  - Lines 186-230: SDP manipulation to force H.264 encoding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liulb/mediaMTX](https://github.com/liulb/mediaMTX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->

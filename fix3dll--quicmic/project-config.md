---
trigger: always_on
description: This document explains the architecture, design choices, key implementation details, and critical guardrails of the **QuicMic** project.
---

# Developer & Agent Guide: QuicMic

This document explains the architecture, design choices, key implementation details, and critical guardrails of the **QuicMic** project. 

---

## 🎙️ Project Overview

**QuicMic** turns a mobile or desktop device's web browser (iOS Safari, Android Chrome, desktop browsers, etc.) into a low-latency wireless PC microphone over a Local Area Network (LAN). 

It consists of:
1. **Frontend (Client):** An HTML5/JS web page that captures mic input via `getUserMedia`, processes it in an `AudioWorklet`, and streams it.
2. **Backend (Server):** A Rust server (using Axum and WTransport) that hosts the web assets, validates client pairing, receives raw audio packets, and plays them into a virtual audio device (such as VB-Cable).

---

## 🛠️ System Architecture & Audio Pipeline

```text
+-------------------------------------------------------------------------+
|                         CLIENT (Mobile / Desktop Browser)               |
|                                                                         |
|    [ Microphone ] ---> ( AudioWorklet ) ---> [ Network Writer ]         |
|     (mono, ~48kHz)       (worklet.js)             (app.js)              |
+-------------------------------------+-----------------------------------+
                                      |
                                      v
+-------------------------------------+-----------------------------------+
|                            LOCAL NETWORK (LAN)                          |
|                                                                         |
|        /--- [ WebTransport ] (UDP/QUIC - Primary Low-Latency) ---\      |
|       /                                                           \     |
|      /---- [ WebSocket ]   (TCP - Secondary Fallback) -------------\    |
+-----+---------------------------------------------------------------+---+
      |                                                               |
      v                                                               v
+-----+---------------------------------------------------------------+---+
|                    SERVER (Windows / macOS / Linux)                     |
|                                                                         |
|    [ wtransport Connection ]              [ axum WebSocket Connection ] |
|               \                                      /                  |
|                v                                    v                   |
|              +----------------------------------------+                 |
|              |   Decode i16 PCM into Ring Buffer      |                 |
|              +--------------------+-------------------+                 |
|                                   |                                     |
|                                   v                                     |
|              +----------------------------------------+                 |
|              |    Ring Buffer (audio/ring_buffer)     |                 |
|              +--------------------+-------------------+                 |
|                                   |                                     |
|                                   v                                     |
|              +----------------------------------------+                 |
|              |           CPAL Output Thread           |                 |
|              +--------------------+-------------------+                 |
|                                   |                                     |
|                                   v                                     |
|              +----------------------------------------+                 |
|              | Catmull-Rom Cubic Resampler (Rate)     |                 |
|              +--------------------+-------------------+                 |
|                                   |                                     |
|                                   v                                     |
|              +----------------------------------------+                 |
|              |    Channel Duplication (Mono -> Multi) |                 |
|              +--------------------+-------------------+                 |
|                                   |                                     |
|                                   v                                     |
|                      [ Virtual Device / Speakers ]                      |
+-------------------------------------------------------------------------+
```

### 1. The Audio Path
- **Client Side:** Capture is mono at the browser's native sample rate (typically 48kHz; the actual rate is reported to the server via the `sr` parameter so the output resampler can match it). The `AudioWorklet` (`web/worklet.js`) buffers samples cleanly and sends them to the main thread.
- **Network Packet Format:** Each packet has a 4-byte little-endian sequence number followed by raw `i16` PCM samples (up to 480 samples = 10ms at 48kHz).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fix3dll/QuicMic](https://github.com/Fix3dll/QuicMic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->

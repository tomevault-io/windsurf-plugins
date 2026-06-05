---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DetectX is an ACAP (Axis Camera Application Platform) application that runs custom YOLOv5 object detection models directly on Axis network cameras with ARTPEC-8 and ARTPEC-9 chipsets. The application performs real-time inference on-camera and exports detections via MQTT, ONVIF, and HTTP.

This is a native C application that integrates with:
- Axis ACAP SDK (version 12.x, native SDK)
- larod (Axis ML inference framework for DLPU hardware acceleration)
- VDO (Video Device Object) for camera video capture
- Paho MQTT for messaging
- FastCGI for web UI backend

## Building and Development

### Build Commands

**Build the entire application:**
```bash
./build.sh
```
This creates `.eap` files (Axis application packages) in the root directory, ready for installation on cameras. Model parameters are automatically extracted during the build process.

**Clean build artifacts:**
```bash
cd app && make clean
```

### Build Process

1. `build.sh` uses Docker with the Axis ACAP SDK image (`axisecp/acap-native-sdk:12.5.0`)
2. The Dockerfile copies `app/` contents and runs `acap-build` to compile and package
3. Output is copied back as `.eap` files (one for ARTPEC-8, one for ARTPEC-9 if multi-architecture build)
4. The Makefile in `app/` compiles all C sources and links against ACAP SDK libraries

### Key Build Dependencies

The Makefile links against these packages (via pkg-config):
- `gio-2.0`, `gio-unix-2.0` - GLib I/O and event loop
- `liblarod` - Axis ML inference library
- `vdostream` - Video capture
- `fcgi` - FastCGI for HTTP endpoints
- `axevent` - Axis event system
- `libcurl` - HTTP client for posting crops

Also links statically: `libjpeg`, `libturbojpeg` (in `app/lib/`)

## Architecture

### Core Components

**main.c**
- Application entry point and main event loop
- `ImageProcess()`: Main inference loop that captures frames, runs inference, applies filters (AOI, confidence, size), and calls Output()
- Manages application configuration (settings.json, events.json) through ACAP wrapper
- Interfaces with all other modules

**ACAP.c/h**
- Wrapper around Axis ACAP SDK (version 3.7)
- Handles HTTP/FastCGI endpoints for web UI backend (registered via `ACAP_HTTP_Node()`)
- Manages persistent JSON configuration files
- Provides device information and status API
- Event system integration (ONVIF events for detection states)

**Model.c/h**
- Sets up and runs neural network inference using larod
- `Model_Setup()`: Initializes model, extracts parameters from TFLite file, allocates larod tensors
- `Model_Inference()`: Preprocesses YUV/RGB video frames, runs inference, performs NMS (non-maximum suppression), returns cJSON array of detections
- `Model_GetImageData()`: Provides JPEG-encoded crops of detections with configurable borders
- `Model_Reset()`: Clears crop cache between inference cycles

**Video.c/h**
- Abstracts VDO (Video Device Object) frame capture
- `Video_Start_YUV()` / `Video_Start_RGB()`: Initialize video streams
- `Video_Capture_YUV()` / `Video_Capture_RGB()`: Capture single frames
- Returns `VdoBuffer*` for processing by Model module

**Output.c/h** (split into multiple files)
- `Output.c`: Main entry point, processes filtered detections
- `Output_crop_cache.c`: Manages in-memory cache of recent detection crops for web UI
- `Output_http.c`: Posts detection crops to external HTTP endpoints
- `Output_helpers.c`: Event state management (transition debouncing, minimum duration)
- Publishes detections and events to MQTT
- Manages ONVIF event states per label

**MQTT.c/h**
- Paho MQTT client wrapper (version 2.0)
- Handles connection, reconnection, publishing
- `MQTT_Publish()`, `MQTT_Publish_JSON()`, `MQTT_Publish_Binary()`: Publish to topics
- Topics: `{pretopic}/detection/{serial}`, `{pretopic}/event/{serial}/{label}/{state}`, `{pretopic}/crop/{serial}`

**CERTS.c/h**
- Certificate management for MQTT TLS connections
- Stores CA certificates, client certificates, and keys

### Configuration Files

**app/manifest.json**
- ACAP package metadata: name, version, vendor
- HTTP endpoint definitions (FastCGI nodes: app, settings, status, device, model, mqtt, certs, crops)

**app/model/model.tflite**
- TFLite model file with int8 quantization
- Model parameters are automatically extracted at runtime by Model.c
- Supported formats: YOLOv5 with per-tensor quantization

**app/model/labels.txt**
- Text file with one label per line
- Loaded at runtime by labelparse.c
- Order must match model output classes

**app/settings/settings.json**
- Detection parameters: confidence threshold, AOI (area of interest), minimum size
- Event settings: stabilization time, minimum event duration, prioritize (accuracy vs responsiveness)
- Cropping configuration: active, throttle, output methods (MQTT/HTTP/SD), border adjustments

**app/settings/mqtt.json**
- MQTT broker connection: address, port, username, password
- TLS configuration
- Topic prefix (pretopic)
- Device metadata: name, location

**app/settings/events.json**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pandosme/DetectX](https://github.com/pandosme/DetectX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->

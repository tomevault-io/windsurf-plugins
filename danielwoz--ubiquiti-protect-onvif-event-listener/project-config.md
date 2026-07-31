---
trigger: always_on
description: This file describes the purpose, structure, and library usage of the ONVIF
---

# AGENTS.md — Codebase guide for AI agents

This file describes the purpose, structure, and library usage of the ONVIF
event recorder project so that AI coding agents can navigate it effectively.

---

## What this project does

The recorder bridges third-party ONVIF IP cameras into a UniFi Protect
installation.  At runtime it:

1. Reads camera credentials from the UniFi Protect PostgreSQL database
   (`cameras` table, `isThirdPartyCamera = true`).
2. Opens a WS-PullPoint subscription to each camera over HTTP/SOAP.
3. Translates raw ONVIF detection events (human / vehicle) into SQLite rows
   that mirror the UniFi Protect `events` and `smartDetectObjects` schema.
4. Optionally fetches a JPEG snapshot from each camera on detection start and
   appends it to a per-camera `.ubv` thumbnail file.
5. Writes every raw SOAP exchange to a timestamped `.jsonl` file for
   replay-based testing.

---

## Source files

| File | Purpose |
|------|---------|
| `src/main.cpp` | Entry point. Loads cameras from UniFi Protect DB, wires up `OnvifListener` + `DetectionRecorder` + `MotionPoller`, handles SIGINT/SIGTERM. Auto-downloads NanoDet-M models. |
| `src/onvif_listener.hpp/.cpp` | `onvif::OnvifListener` — manages WS-PullPoint subscriptions; one thread per camera. Parses SOAP XML into `OnvifEvent` structs and delivers them via a callback. |
| `src/detection_recorder.hpp/.cpp` | `onvif::DetectionRecorder` — filters events to human/vehicle/animal/package detections, maintains open-event state, writes to PostgreSQL, fetches snapshots via libcurl, crops thumbnails via NanoDet-M or ONVIF bbox. |
| `src/alarm_notifier.hpp/.cpp` | `onvif::AlarmNotifier` — triggers Protect automations on smart detection events via the local Protect API (port 7080). Records automation history with cooldown support. |
| `src/motion_poller.hpp/.cpp` | `onvif::MotionPoller` — polls the events table for first-party camera `motion` events, runs NanoDet-M on Protect thumbnails, and inserts smart detection records. |
| `src/camera_change_log.hpp/.cpp` | `unifi::CameraChangeLog` — thread-safe JSON Lines log of cameras-table mutations (old/new values). Used for rollback support. |
| `src/protect_ui_patch.hpp/.cpp` | `protect_ui::patch_alarm_picker()` / `revert_alarm_picker()` — live-patches the Protect UI (swai*.js, vantage*.js, service.js) so third-party cameras appear in alarm creation. Uses dpkg md5sums for backup integrity. |
| `src/ubv_thumbnail.hpp/.cpp` | `ubv::encode` / `ubv::decode` / `ubv::append` — minimal UBV container for storing JPEG thumbnail frames. |
| `src/jpeg_crop.hpp/.cpp` | JPEG decode/crop/re-encode via libjpeg-turbo. Used for thumbnail cropping from ONVIF bounding boxes and NanoDet-M detections. |
| `src/object_detect.hpp/.cpp` | `onvif::ObjectDetector` — NanoDet-M on-device object detection via NCNN. Returns bounding boxes and COCO class labels. Built with NEON SIMD on ARM64. |
| `src/event_recorder.hpp/.cpp` | `onvif::EventRecorder` — thread-safe JSON Lines writer for parsed ONVIF events. Used by `--event_log`. |
| `src/util.hpp/.cpp` | `onvif::util::` — shared helpers: UUID v4 generation, 24-char hex ID generation, ISO-8601 timestamps, JSON string escaping. |
| `src/unifi_camera_config.hpp/.cpp` | `unifi::load_cameras()` — queries the UniFi Protect PostgreSQL instance and returns a `CameraConfig` per adopted third-party camera. Also handles smart-detect flag enablement, rollback, and first-party camera discovery. |
| `src/ubv_extract.cpp` | Standalone tool to extract JPEG frames from UBV thumbnail files. |
| `test/onvif_camera_emulator.hpp/.cpp` | HTTP server (libmicrohttpd) that replays raw `.jsonl` SOAP logs; used as a fake camera in tests. |
| `test/camera_emulators.hpp/.cpp` | Concrete emulators for Camera 108 (FieldDetector) and Camera 109 (UserAlarm/IVA). |
| `test/test_onvif_listener.cpp` | Drives `OnvifListener` against emulated cameras; JSONL path passed automatically by Bazel. |
| `test/test_detection_recorder.cpp` | End-to-end test: emulated camera → `DetectionRecorder` → PostgreSQL-like assertions. |
| `test/test_ubv_thumbnail.cpp` | Round-trip test: encodes snapshot JPEGs into a UBV file, decodes, verifies fidelity. |
| `test/test_camera_change_log.cpp` | CameraChangeLog write/read roundtrip, concurrent writes, malformed line recovery. |
| `test/test_protect_ui_patch.cpp` | Protect UI patch apply/revert logic, dpkg md5sum backup validation. |
| `test/test_unifi_camera_config.cpp` | DB connection string / JSON extraction / PG array literal helper tests. |
| `test/test_motion_poller.cpp` | Smart detect type mapping / SDR payload JSON generation tests. |
| `test/testdata/` | Test fixtures: snapshot JPEGs, JSONL replay logs for multiple camera brands. |

---

## Key types

### `onvif::OnvifEvent`
Delivered to the `EventCallback` for every received event:
```cpp
struct OnvifEvent {
    std::string camera_ip;
    std::string camera_user;
    std::string topic;        // e.g. "tns1:RuleEngine/FieldDetector/ObjectsInside"
    std::string event_time;   // camera-reported UTC timestamp
    std::string property_op;  // "Initialized", "Changed", or "Deleted"
    std::map<std::string, std::string> source;
    std::map<std::string, std::string> data;
};
```

### `onvif::CameraConfig`
```cpp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielwoz/ubiquiti-protect-onvif-event-listener](https://github.com/danielwoz/ubiquiti-protect-onvif-event-listener) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

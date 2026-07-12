---
trigger: always_on
description: - Board: JC-ESP32P4-M3-DEV (ESP32-P4, ES8311 codec, SD card).
---

AGENT NOTES
===========

Context
-------
- Board: JC-ESP32P4-M3-DEV (ESP32-P4, ES8311 codec, SD card).
- Wake word: WakeNet9, model `wn9_hiesp`, 16 kHz mono, threshold 0.50-0.95 (runtime adjustable).
- HA Assist pipeline via WebSocket (`ws://<HA_IP>:9000/api/websocket`), MQTT broker `mqtt://<HA_IP>:1883`.
- OTA binary: `build/esp32_p4_voice_assistant.bin`.
- OTA URL (local server): `http://<PC_IP>:8080/build/esp32_p4_voice_assistant.bin` (`ota_server.bat` prints the URL).

Recent fixes (Dec 25, 2025)
---------------------------
### Music Persistence & Reliability (v0.2.0)
- **Local Music Player**:
    - Fixed race condition where "Play Music" command failed if SD card wasn't fully ready at boot.
    - Added auto-initialization check: if player is not ready when "Play Music" is called, it attempts to initialize immediately.
- **Voice Pipeline**:
    - Increased `pipeline_cmd_queue` send timeout from 0 to 100ms to prevent dropping critical commands (like STOP_WWD) when the system is busy.
    - Improved robustness of music pipeline switching logic.

### Voice-Triggered Music Fix (v0.2.1)
- **Voice Pipeline**:
    - Fixed issue where voice-triggered "Play Music" command would hang the system.
    - Implemented `PIPELINE_CMD_MUSIC_CONTROL` to safely stop the microphone and wait for resource release before starting music playback.

### Stability Improvements (v0.2.2)
- **Voice Pipeline**:
    - Increased `pipeline_task` stack size from 4KB to 8KB to prevent stack overflow crashes during music playback (file I/O + codec operations).
    - Added watchdog feeding (`sys_diag_wdt_feed()`) during heavy music initialization steps to prevent WDT resets.

### Critical Stability Fix (v0.2.3)
- **Voice Pipeline**:
    - Further increased `pipeline_task` stack size to 12KB. 8KB proved insufficient for simultaneous file operations and text-to-speech handling.
    - Added `audio_capture_stop_wait` during initialization to ensure clean audio state.

### Music/TTS Conflict Fix (v0.2.4)
- **Voice Pipeline**:
    - Implemented TTS suppression when music command is detected. This prevents Home Assistant's TTS ("Playing music") from conflicting with the music player's codec usage, which was causing the device to hang/block.

### Internal RAM Fix (v0.2.5)
- **Voice Pipeline**:
    - Moved `pipeline_task` stack allocation from internal RAM to PSRAM. The 12KB stack was exhausting internal RAM, causing MQTT/network failures ("out of memory", "thread_sem_init: out of memory").

Recent fixes (Dec 24, 2025)
---------------------------
### Comprehensive Bug Fixes (v0.1.9)
- **Critical Memory Safety**: 
    - Fixed use-after-free in `fetch_task` (moved cleanup to `audio_capture_stop_wait`).
    - Fixed memory leaks in `voice_pipeline.c` (`current_pipeline_handler`), `ha_client.c` (`audio_frame_buf`), and `ha_client_start_conversation` (missing malloc check).
    - Fixed race conditions with `callback_mux` spinlock in `ha_client.c` and `is_running` flag in `audio_capture.c`.
    - Added missing null checks for `cJSON_PrintUnformatted` and `malloc`.
- **Functionality**:
    - **AGC**: Stub functions now correctly return `false`/`ESP_ERR_NOT_SUPPORTED` instead of misleading success.
    - **TTS**: Buffer overflow warning now includes size information (`used/max`).
    - **Cleanup**: Proper null termination for `strncpy` in MQTT handlers.
- **Code Quality**:
    - Removed redundant `extern` declarations from `tts_player.c` and `beep_tone.c`.
    - Replaced magic numbers with named constants (`BEEP_WAKE_*`, `BEEP_CONFIRM_*`, `BEEP_ERROR_*`).
    - Removed conflicting "Local Time Question" feature (now handled via LLM system prompt).

Recent fixes (Dec 20, 2025)
---------------------------
### OTA + WebSerial reliability
- OTA now validates HTTP status, supports unknown `Content-Length`, logs errors clearly, and uses PSRAM stack fallback if needed.
- WebSerial HTTP header limit raised to 8192 to avoid `431 Request Header Fields Too Large` on large requests.
- OTA start failures now log error names for faster diagnosis.

### MQTT + HA entities
- `sw_version` uses `esp_app_get_description()->version` (include `esp_app_desc.h`).
- WebSerial metric renamed to `webserial_requests` (it counts log requests, not active clients).
- Legacy MQTT discovery cleanup clears old/duplicated entities on connect; update `legacy_discovery_topics` when renaming/removing entity IDs.
- Added `diagnostic_dump` button (MQTT) to emit `sys_diag_report_status`.

### Audio behavior
- Timer/alarm beep now plays at max volume and then restores the previous output volume.
- Guard against empty response text before indexing in voice pipeline response handling.

Recent fixes (Dec 17, 2025)
---------------------------
### LED Status Improvements
- **SPEAKING LED**: Cyan fast pulsing (300ms period) triggered on `tts-start` event, not `tts-end`.
- **OTA LED**: White fast pulsing (300ms period) during firmware updates.
- LED status log now includes task name for debugging: `LED status: X -> Y [task_name]`.
- Fixed LED timing: LED is set to SPEAKING *before* TTS download starts, stays cyan during playback.

### WWD Threshold Runtime Control
- Added `wwd_set_threshold(float threshold)` and `wwd_get_threshold()` to `wwd.c/wwd.h`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvucinozd/Home-Assistant-MQTT-Voice-Assistant](https://github.com/dvucinozd/Home-Assistant-MQTT-Voice-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->

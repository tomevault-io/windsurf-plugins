---
trigger: always_on
description: Guidance for AI coding agents working on this repository. Assumes no prior knowledge of the project.
---

# AGENTS.md

Guidance for AI coding agents working on this repository. Assumes no prior knowledge of the project.

## Project Overview

**ESP32MQTTClient** is a C++ MQTT client library for ESP32, built directly on the official ESP-IDF `esp-mqtt` component (`mqtt_client.h`) rather than PubSubClient. It works in two environments:

- **Arduino ESP32** (`arduino-esp32` v2/v3+), distributed as an Arduino library (`library.properties`, currently version 1.1.3, `architectures=esp32`).
- **Native ESP-IDF** (v4.x and v5.x), registered as an ESP-IDF component via the top-level `CMakeLists.txt` / `component.mk`.

Key characteristics:

- **Non-blocking**: `loopStart()` returns immediately; MQTT runs in a background FreeRTOS task managed by esp-mqtt. No `loop()` polling call is required in user code.
- Uses standard C++ `std::string` everywhere — **never** Arduino `String`.
- Logging uses ESP-IDF `ESP_LOGX` macros with tag `"ESP32MQTTClient"`.
- Interfaces are inspired by [EspMQTTClient](https://github.com/plapointe6/EspMQTTClient).
- TLS/SSL support via `setCaCert()` / `setClientCert()` / `setKey()`; `setURL()` automatically selects the `mqtts://` scheme when port is 8883.
- Supports per-topic subscription callbacks, a global catch-all callback (`setOnMessageCallback`), MQTT wildcards (`#`, `+`) matched by `mqttTopicMatch()`, and reassembly of fragmented incoming messages (`MQTT_EVENT_DATA` chunks buffered in `_incomingTopic`/`_incomingPayload` until complete).

## Repository Layout

```
src/ESP32MQTTClient.h        # Public API + class definition (only library header)
src/ESP32MQTTClient.cpp      # Full implementation
CMakeLists.txt               # ESP-IDF component registration (idf_component_register, REQUIRES mqtt)
component.mk                 # Legacy make-based ESP-IDF component file
library.properties           # Arduino Library Manager metadata
keywords.txt                 # Arduino IDE syntax highlighting
examples/HelloToMyself/      # Arduino sketch example (.ino)
examples/CppEspIdf/          # Native ESP-IDF example project
  components/ESP32MQTTClient/CMakeLists.txt  # Thin wrapper that compiles ../../../../src directly
.github/workflows/           # CI (see Testing / CI below)
README.md                    # User-facing documentation and API reference
```

The entire library is a single class (`ESP32MQTTClient`) in one header/source pair. Keep it that way unless there is a strong reason to split.

## Build and Test Commands

There are no unit tests in this repository. Verification is compile-based, via the two CI workflows and the example projects.

### Arduino ESP32 (CI)

`.github/workflows/ci4main.yml` uses [adafruit/ci-arduino](https://github.com/adafruit/ci-arduino): it checks out that repo into `ci/`, runs `bash ci/actions_install.sh`, then `python3 ci/build_platform.py esp32`, which compiles the `.ino` examples against arduino-esp32. Note: `ci/build_platform.py` is **not** in this repo — it comes from the checked-out ci-arduino repo. Locally, equivalent checks are:

```bash
# With arduino-cli (ESP32 core installed):
arduino-cli compile --fqbn esp32:esp32:esp32 examples/HelloToMyself
```

### Native ESP-IDF

```bash
# Set up the ESP-IDF environment first (tested versions: v4.4.6 and v5.3):
export IDF_PATH=/path/to/esp-idf
. $IDF_PATH/export.sh

cd examples/CppEspIdf
idf.py build                  # build
idf.py flash monitor          # flash and watch serial output
```

Before building the example, edit `examples/CppEspIdf/main/main.cpp` to set real `WIFI_SSID`/`WIFI_PASS` (and broker URI) values — the committed values are placeholders.

### CI matrix

`.github/workflows/esp_idf_ci.yml` builds `examples/CppEspIdf` with `espressif/esp-idf-ci-action` for target `esp32` against **ESP-IDF v4.4.6 and v5.3**. Any change to `src/` must compile under both. Note that the IDF CI compiles `src/` through the thin wrapper in `examples/CppEspIdf/components/ESP32MQTTClient/`; the top-level `CMakeLists.txt` component-registration path is not directly covered by CI.

## Code Style Guidelines

### Naming
- Classes: `PascalCase` (`ESP32MQTTClient`, `TopicSubscriptionRecord`)
- Methods / locals: `camelCase` (`loopStart`, `payloadStr`)
- Private members: leading underscore (`_mqtt_client`, `_mqttConnected`, `_topicSubscriptionList`)
- Constants / macros: `UPPER_SNAKE_CASE` (`WIFI_SSID`, `TAG`, `DEFAULT_PACKET_SIZE`)
- Typedefs: `PascalCase` (`MessageReceivedCallback`, `MessageReceivedCallbackWithTopic`)

### Types and strings
- `std::string` only; pass as `const std::string &`. Convert Arduino `String` at call sites with `.c_str()`.
- `uint16_t` / `uint8_t` for sizes and QoS, `int` for esp-mqtt return codes, `bool` for success/failure.
- `publish()` passes the full `std::string` length to esp-mqtt, so binary payloads with embedded `\0` are preserved — keep it that way.
- Inline trivial getters/setters in the header (see `isConnected()`, `setURI()`, `setURL()`).

### Compilation constraints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyijun/ESP32MQTTClient](https://github.com/cyijun/ESP32MQTTClient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

---
trigger: always_on
description: Instructions for AI coding agents working in this repository: a device-side C SDK for Tuya
---

# AGENTS.md

Instructions for AI coding agents working in this repository: a device-side C SDK for Tuya
hardware, spanning three bounded contexts — an IoT client (device ↔ cloud over MQTT/ATOP), an
RTC/TAI client (device ↔ AI Foundation, real-time audio over a custom binary framing), and BLE
provisioning — over a Platform Abstraction Layer, with mbedTLS / cJSON / coreMQTT / coreHTTP
vendored under `third_party/`. Sections follow the team outline.

## Commands

```bash
git submodule update --init --recursive          # mbedTLS nests a `framework` submodule
pip install jsonschema jinja2 pycryptodome cryptography
cmake -S . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DPython3_EXECUTABLE=$(which python3)
cmake --build build -j 4
ctest --test-dir build --output-on-failure --no-tests=error --timeout 180   # never -j
```

- **One test**: `ctest --test-dir build -R iot_dp_test --output-on-failure`, or
  `./build/iot_dp_test` from any cwd — every mock path is an absolute compile-time define.
- **Sanitizers** — the only memory-error gate in CI:
  `cmake -S . -B build-asan -DCMAKE_BUILD_TYPE=Debug -DPython3_EXECUTABLE=$(which python3) -DCMAKE_C_FLAGS="-fsanitize=address,undefined -fno-sanitize-recover=all -g -O1" -DCMAKE_EXE_LINKER_FLAGS="-fsanitize=address,undefined"`
  then `ASAN_OPTIONS=detect_leaks=0 UBSAN_OPTIONS=halt_on_error=1 ctest --test-dir build-asan --output-on-failure --no-tests=error --timeout 240`
- **Leaks**: `modules/iot-client/test/run_valgrind_check.sh ./build/tai_unit_tests` (Linux) or
  `run_leaks_check.sh ./build/iot_cipher_test` (macOS) — no-subprocess binaries only; under
  valgrind the Python mock handshakes time out. Both scripts **ignore the test's exit code**: a
  suite failing every assertion still prints "RESULT: NO MEMORY LEAKS DETECTED" and exits 0.
  Coverage is 3 of 13 binaries, hand-listed in `.gitlab-ci.yml`; GitHub's sanitizer job sets
  `detect_leaks=0`, so nothing else checks leaks anywhere.
- **Examples** — a separate top-level project, and `AGENTIC_KIT_BUILD_EXAMPLES` defaults OFF, so
  a plain root `cmake --build build` builds none:
  `cmake -S examples/posix -B build-examples -DPython3_EXECUTABLE=$(which python3) && cmake --build build-examples -j 4`
  Three demos then remove themselves on a `message(STATUS)` alone — `chat_demo` /
  `edu_camera_demo` without a matching prebuilt `libstm.a`, `tai_audio_chat_demo` without
  libopus — so confirm the specific target actually got built.
- **Docs site**: `cd docs-site && npm ci && npm run build` (node ≥ 20).
- **Version**: `tools/bump_version show | next [--major|--minor|--patch] | release` — README's
  `tools/bump_version.sh` and `tools/bump_version_test.sh` do not exist; the script has no
  extension and no separate self-test.

## Architecture

- **Two threading models, and the app owns only one loop.** iot-client: single-threaded,
  app-driven — nothing runs unless the app calls `iot_client_process()`, and every DP / message /
  reset callback fires on that thread. rtc-tcp-client: `tai_connect()` spawns a worker via
  `pal->thread_create`, and every receive callback fires there, concurrently with the app.
  tuya-ble: no thread — it runs on whatever context the port's BLE stack calls `tuya_ble_recv()`
  from. Never move work between these worlds.
- **The SDK does no background work.** It never auto-reconnects MQTT, and the DP layer never
  publishes on its own — no report-on-connect, no cloud-query-triggered report
  (`modules/iot-client/docs/adr/0001-dp-layer-never-initiates-uplink.md`). The app owns the
  connect/reconnect loop and every uplink.
- **`modules/rtc-client` is a prebuilt closed-source library**: four headers plus one `libstm.a`
  per architecture — no source, no CMakeLists, no test. Call its API, never change it; no change
  to it is verifiable offline. The root CMakeLists declares `tuya_steam_client` only when the
  current platform's `libstm.a` exists (its if/elseif chain covers three of the five
  architectures on disk).
- **coreMQTT and coreHTTP route their Error/Warn logs into the log facade** via
  `common/core_mqtt_config.h` and `common/core_http_config.h`, and must keep doing so. Setting
  `MQTT_DO_NOT_USE_CUSTOM_CONFIG` / `HTTP_DO_NOT_USE_CUSTOM_CONFIG` again silently discards each
  library's own account of a failure: a rejected CONNECT decays to a bare `MQTTServerRefused` with
  no `Connection refused: bad user name or password.`, and an oversized response to a generic
  communication error with no `insufficient space`. Both build paths must be kept in step — the
  root `CMakeLists.txt` **and** `examples/esp-idf/components/agentic_kit`, which carries its own
  `target_compile_definitions` and was for a while the path where none of this reached a device.
  Pinned by `test_connack_reason_is_logged` and `test_oversized_response_is_explained`.
  `LogInfo`/`LogDebug` stay compiled out on purpose (per-packet, costly on flash), and coreMQTT's
  thread-safety hooks are still unset.

## Conventions

1. **Domain language is not optional.** `CONTEXT-MAP.md` points at one `CONTEXT.md` per context;
   each defines the terms *and* the words to avoid. Use those exact words in code, commit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuya/agentic-kit](https://github.com/tuya/agentic-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

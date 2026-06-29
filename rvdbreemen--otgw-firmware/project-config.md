---
trigger: always_on
description: These instructions adapt the repository's Claude guidance for Codex agents. They are project-specific rules for this firmware repository and sit above the generic Backlog.md CLI reference below.
---

# OTGW-firmware: Codex Agent Instructions

These instructions adapt the repository's Claude guidance for Codex agents. They are project-specific rules for this firmware repository and sit above the generic Backlog.md CLI reference below.

---

## Task Management

Every meaningful code or documentation change should be backed by a Backlog task before implementation work starts. Use Backlog MCP tools as the primary interface for reading, creating, updating, and completing tasks.

Codex agents must not edit files in `backlog/tasks/` directly. Use Backlog MCP tools such as task view/search/create/edit when available. Use the Backlog CLI commands in the reference below only when the MCP tools are unavailable, missing required capability, or fail for the current operation.

Before marking a task `Done`, run through `docs/guides/pr-checklist.md`. A clean build is the minimum bar; the checklist covers hardware, browser, MQTT, and smoke-test expectations that compile checks do not cover.

After marking a task `Done`, commit the completed task changes and push the branch to its configured upstream. Keep commits narrow: stage only files that belong to the completed task and leave unrelated local changes untouched.

Known issue: `backlog task list` may return empty in this repository. Prefer Backlog MCP search/view; if falling back to CLI, prefer `backlog search "<topic>" --plain` or `backlog task <id> --plain`. Read task files directly only as a read-only fallback, never to modify them.

---

## Design Principles

- KISS: choose the simplest solution that works and explain tradeoffs when complexity is optional.
- YAGNI: do not add behavior for hypothetical future needs.
- Minimal change surface: keep changes small, focused, and justified by the task.
- Comments describe the present system. Avoid defensive comments about hypothetical modes or future work; if the concern is real, create a Backlog task.
- Fix the documentation before renaming identifiers when the identifier is still semantically correct and only the comment or docstring is stale.

---

## Project Overview

This repository contains ESP8266 and ESP32 firmware for the NodoShop OpenTherm Gateway, including Web UI, MQTT, REST API, TCP serial bridge, Home Assistant integration, OTGW32 support, and SAT-related work.

- Platform: ESP8266 (NodeMCU/Wemos D1 mini, tight RAM budget) and ESP32/OTGW32.
- Language: Arduino C/C++ in `.ino` files, built as a single translation unit.
- Serial: reserved for the PIC after initialization. Never write to `Serial` after OTGW init.
- Debug: use `DebugTln()`, `DebugTf()`, and related debug helpers. They go to Telnet port 23; do not use `Serial.print()`.
- Branches (model changed 2026-06-20): `dev` is the DEFAULT line and carries the 2.0.0 ESP32-S3-only async + FreeRTOS firmware (epic TASK-865; ESP8266 dropped; former `feature-2.0.0-esp32s3-async`). `otgw-1.x.x` is the 1.5.x/1.6.x maintenance/LTS line (the former `dev`). `main` is ALWAYS the latest public release. Default to the current branch and port fixes between `dev` and `otgw-1.x.x` deliberately.

---

## Critical Coding Rules

### PROGMEM strings

Keep string literals in flash wherever practical:

```cpp
DebugTln(F("Message"));
DebugTf(PSTR("Value: %d\r\n"), value);
snprintf_P(buf, size, PSTR("fmt: %s"), str);
```

Use `strcmp_P()` and `strcasecmp_P()` with `PSTR()` for flash-resident string comparisons. Use `memcmp_P()` for binary data; do not use `strncmp_P()` or `strstr_P()` on binary payloads.

### JSON

Do not add ArduinoJson. This firmware builds JSON manually with `snprintf_P()` / `sendJsonMapEntry()` and parses with `parseJsonKVLine()`.

### String usage

Avoid Arduino `String` in hot paths, especially areas covered by ADR-004: SAT, MQTT, REST, OTGW-Core, and OTDirect. Prefer fixed buffers with `strlcpy()` and `snprintf_P()`. `String` is acceptable only in setup or one-off paths where the local pattern already allows it.

### File serving

Stream files instead of loading them into RAM. Files larger than roughly 2 KB should use `httpServer.streamFile()`. `index.html` is too large for `readString()` style handling.

### HTTP and WebSocket

Do not add HTTPS or WSS support in firmware. This project targets trusted LAN deployment; REST can sit behind an HTTPS reverse proxy, while WebSocket assumptions remain plain WS.

### ESP Platform Abstraction

The 2.0.0 branch carries an explicit platform abstraction. **No `#if(def) ESP8266`, `#if(def) ESP32`, `#if(def) ARDUINO_ARCH_ESP*`, or `#if(def) BOARD_NODOSHOP_ESP*` may appear outside the allowlisted abstraction files.** The allowlist is: `src/OTGW-firmware/platform.h`, `platform_esp8266.h`, `platform_esp32.h`, `boards.h`, and the `OTGW-ModUpdateServer{.h,-esp32.h,-impl.h}` trio.

Application code must instead:

- Call `platformXxx()` shims from `platform_*.h` for any divergent API (heap, hostname, MAC, reset, NTP, LED, JSON tx, WiFi, BLE, ...). If the shim you need does not exist, **add it first** in both `platform_esp8266.h` and `platform_esp32.h` (use an inline no-op stub on the platform where the feature is absent), then call it unguarded from application code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rvdbreemen/OTGW-firmware](https://github.com/rvdbreemen/OTGW-firmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

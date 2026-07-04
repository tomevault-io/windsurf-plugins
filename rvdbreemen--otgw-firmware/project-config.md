---
trigger: always_on
description: This is the ESP8266 firmware for the NodoShop OpenTherm Gateway (OTGW). It provides network connectivity (Web UI, MQTT, REST API, and TCP serial socket) for the OpenTherm Gateway hardware, with a focus on reliable Home Assistant integration.
---

# GitHub Copilot Instructions for OTGW-firmware

## Project Overview

This is the ESP8266 firmware for the NodoShop OpenTherm Gateway (OTGW). It provides network connectivity (Web UI, MQTT, REST API, and TCP serial socket) for the OpenTherm Gateway hardware, with a focus on reliable Home Assistant integration.

## ADR Workflow Reminder

**IMPORTANT:** This project maintains Architecture Decision Records (ADRs) as docs-as-code that document key architectural choices. Before making changes that affect architecture, consult the relevant ADRs:

- **Platform & Architecture:** See `docs/adr/` directory for complete ADR index
- **Key decisions documented:** ESP8266 platform, modular .ino files, HTTP-only (no HTTPS), static buffers, PROGMEM strings, WebSocket streaming (OpenTherm messages only), MQTT integration, timer-based scheduling, LittleFS persistence, hardware watchdog, PIC firmware upgrade, Arduino framework, build system, NTP/timezone, command queue, WiFiManager, ArduinoJson, simplified OTA flash (XHR-based, see ADR-029)
- **ADR Index:** `docs/adr/README.md` provides navigation and decision summaries
- **ADR Skill:** `.github/skills/adr/SKILL.md` provides comprehensive ADR creation guidance

Treat `docs/adr/README.md` as the **single source of truth** for:
- when an ADR is required
- the ADR template and naming conventions
- lifecycle, immutability, and superseding rules
- PR and code-review expectations for architectural changes

For architecturally significant changes, read the relevant ADRs before coding, follow the existing decisions, and link any applicable ADR in the PR description. Use `.github/skills/adr/SKILL.md` when you need help drafting or checking an ADR.

### ADR Hotspots

The following areas frequently look like "just a bug fix" but often cross into architecture, contracts, or NFRs. When working in these files or subsystems, explicitly ask whether an ADR is affected before coding.

- **OTA / update flow**: `OTGW-ModUpdateServer-impl.h`, `OTGW-ModUpdateServer.h`, `updateServerHtml.h`, `flash_esp.py`, `build.py`
  - Ask: does this change alter update transport, reboot verification, browser/server coordination, persistence behavior, or operator recovery workflow?
- **Settings / state model**: `OTGW-firmware.h`, `settingStuff.ino`, settings persistence helpers, boot-time settings load
  - Ask: does this change alter the configuration model, runtime state ownership, initialization ordering, or persistence format?
- **Persistence and filesystem behavior**: `LittleFS` usage, `settings.ini`, `version.hash`, streaming file serving, backup/restore during flash
  - Ask: does this change alter what survives reboot/flash, how files are written, or memory-safety patterns for file handling?
- **REST API and external contracts**: `restAPI*.ino`, JSON payload shapes, endpoint routing, HTTP status behavior
  - Ask: does this change alter an API contract, response structure, versioning behavior, or compatibility expectations?
- **MQTT and Home Assistant integration**: `MQTTstuff.ino`, topic naming, discovery payloads, source-specific topic layout
  - Ask: does this change alter published topics, retained payload structure, discovery entities, or client compatibility?
- **Network behavior and protocol choices**: HTTP/WS communication, WebSocket lifecycle, telnet diagnostics, polling/state machines
  - Ask: does this change alter transport assumptions, security model, concurrency behavior, or service coordination?
- **Build / tooling / release flow**: `build.py`, `Makefile`, evaluation checks, artifact naming, CI/CD behavior
  - Ask: does this change alter developer workflow, build guarantees, release artifacts, or policy enforcement?
- **Memory-safety patterns tied to existing ADRs**: PROGMEM usage, static buffers, file streaming, heap protection, protocol hot paths
  - Ask: is this only an implementation cleanup, or does it change an established project-wide rule?

If the answer is unclear, stop and inspect `docs/adr/README.md` before proceeding. If a change modifies one of these project-wide behaviors, prefer documenting it with an ADR or explicitly recording why no ADR is needed.

## Technology Stack

- **Platform**: ESP8266 (NodeMCU / Wemos D1 mini)
- **Language**: Arduino C/C++ (.ino files)
- **Core Library**: ESP8266 Arduino Core 2.7.4
- **Filesystem**: LittleFS
- **Build System**: Makefile with arduino-cli
- **Key Libraries**:
  - WiFiManager - WiFi configuration and connection management
  - ArduinoJson - JSON parsing and serialization
  - PubSubClient - MQTT client
  - TelnetStream - Debug telnet server
  - AceTime - Time and timezone handling
  - OneWire/DallasTemperature - Dallas temperature sensors

## Architecture

- **Main firmware**: OTGW-firmware.ino (setup and main loop)
- **Modular .ino files**: Each module handles a specific feature (MQTT, REST API, settings, etc.)
- **Communication**: Serial interface to OpenTherm Gateway PIC controller
- **Integration**: MQTT for Home Assistant Auto Discovery, REST API, TCP socket for OTmonitor

## Architecture Decision Records (ADRs)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rvdbreemen/OTGW-firmware](https://github.com/rvdbreemen/OTGW-firmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

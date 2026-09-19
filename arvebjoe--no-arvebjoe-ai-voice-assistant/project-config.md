---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Homey (Athom) SDK v3 app that connects ESP32-based voice devices (Home Assistant Voice PE, XiaoZhi AI) running ESPHome firmware to OpenAI's Realtime API. It bridges on-device mic/speaker with cloud STT/LLM/TTS and lets the LLM control the Homey smart home via tool calls.

## Commands

```bash
npm run build          # tsc -> compiles .mts to .homeybuild/
npm run lint           # eslint (config: athom/homey-app)
npm test               # vitest run (one-shot)
npm run test:watch     # vitest watch mode
npm run test:coverage  # vitest with coverage
npx vitest run tests/weather-helper.test.mts   # run a single test file
```

Running the app on a Homey requires the Homey CLI (not an npm script):
- `homey app run --remote` — **preferred for live debugging.** Runs the app on the real Homey and streams its log back to the terminal so you (and Claude) can follow `this.homey.log(...)` output live. Use this when investigating runtime behavior (e.g. pairing/discovery).
- `homey app run` — live-reload on a real Homey.
- `homey app install` — install the app onto the Homey.

`app.json` is **generated** from `.homeycompose/` by the CLI's build/compose step — edit files under `.homeycompose/` (app metadata, capabilities, flow cards, discovery), never `app.json` directly. The compose step also runs as part of `homey app run`, so changes under `.homeycompose/` (including `discovery/esphome.json`) take effect on the next run.

## Module system gotcha

Source files are `.mts` (ESM TypeScript) compiled by `tsc` to `.homeybuild/`. Imports reference the **compiled** extension, so a file `foo.mts` is imported as `from './foo.mjs'`. Match this convention in every import.

## Architecture

### App bootstrap and singletons (`app.mts`)

`AiVoiceAssistantApp` (the `Homey.App` entry point) constructs the shared services in order and stores them on the app instance: `settingsManager` (init), `GeoHelper`, `WeatherHelper`, `WebServer`, `ApiHelper`, `DeviceManager`. Devices reach these via `(this.homey as any).app.deviceManager` etc. — they are not re-instantiated per device.

### Driver/device inheritance

Both drivers are thin subclasses of shared base classes in `src/homey/`:
- `drivers/home-assistant-voice-preview-edition/` and `drivers/xiaozhi-ai/` each have a `device.mts` + `driver.mts` that extend `VoiceAssistantDevice` / `VoiceAssistantDriver`.
- All real logic lives in `src/homey/voice-assistant-device.mts` and `voice-assistant-driver.mts`. Subclasses only set per-model flags like `needDelayedPlayback` and `thisAssistantType`.
- Flow card run-listeners are registered **once** across all driver instances (guarded by a static `flowCardsInitialized` flag in `VoiceAssistantDriver`).
- **Pairing** (PE + TR drivers): a custom `onPair` flow — a `start` choice view, then either the system `list_devices` (mDNS, backed by `onPairListDevices`) or a **Bluetooth Wi-Fi setup wizard** (`pair/improv_setup.html`, identical copies per driver — keep in sync) that provisions un-networked devices via **Improv over BLE**. Protocol client: `src/ble/improv-ble-client.mts`; pair-socket wiring: `src/ble/improv-pair-handlers.mts` (unit-tested with fakes in `tests/mocks/mock-improv-ble.mts`). Needs the `homey:wireless:ble` permission. Reference: `docs/wifi-provisioning-improv-ble.md`. Manual IP entry (`pair/manual_entry.html`, also identical copies) is the mDNS-less fallback and collects the optional API encryption key. Encrypted devices (mDNS `txt.api_encryption`, or a probe hitting the Noise indicator) are listed marked "needs encryption key" without identity probing; `list_devices` navigates to the `encryption_check` loading view, whose showView handler routes encrypted selections to `manual_entry` with the address prefilled (`manual_get_prefill`) and everything else on to `add_devices`. Gated per driver by `supportsEncryptedPairing` (false for XiaoZhi — its pair flow has no manual_entry view).

### Voice pipeline (the core data flow)

```
ESP32 device  <--TCP/protobuf-->  EspVoiceAssistantClient  <-->  VoiceAssistantDevice  <--WebSocket-->  OpenAIRealtimeAgent
   (ESPHome)        port 6053       (src/voice_assistant/)        (src/homey/)              (src/llm/)        |
                                                                                                          ToolManager
```

- **`src/voice_assistant/esp-voice-assistant-client.mts`** — TCP client speaking the ESPHome native API (protobuf). It handles reconnect/health-check (ping timeout, health interval), emits `chunk` (16kHz PCM audio), `capabilities`, `volume`, `mute`, `started`/`starting` events. `esp-messages.mts` loads `api.proto` via protobufjs and varint-frames messages.
- **`src/homey/voice-assistant-device.mts`** — orchestrates a session: wires the ESP client to the OpenAI agent, resamples mic audio (`Pcm16kTo24k`), segments PCM (`PcmSegmenter`), encodes responses to FLAC (`audio-encoders.mts`) and serves them over LAN HTTP through `WebServer` so the device can play a URL.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arvebjoe/no.arvebjoe.ai-voice-assistant](https://github.com/arvebjoe/no.arvebjoe.ai-voice-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->

---
trigger: always_on
description: A small **wearable OS** in C# (Android-shaped, ESP32-sized) for the **Waveshare ESP32-S3-Touch-AMOLED-2.06 watch**. SpawnWear is not a single firmware app — it's a launcher + system services + UI framework + apps that the user installs/launches/closes.
---

# SpawnWear — Project Instructions

## Mission

A small **wearable OS** in C# (Android-shaped, ESP32-sized) for the **Waveshare ESP32-S3-Touch-AMOLED-2.06 watch**. SpawnWear is not a single firmware app — it's a launcher + system services + UI framework + apps that the user installs/launches/closes.

Flagship app: **AI Assistant** — voice + text chat to TJ's home PC over WebRTC (SpawnDev.RTC), watch is the I/O device. Other built-in apps: Settings, Clock, Media Player, Voice Recorder, Activity.

Companion **Blazor WebAssembly PWA** mirrors the watch UI over BLE + WiFi — useful for setup before the on-screen keyboard is comfortable, and for live debugging from a laptop. **Optional**, never required.

**Primary target hardware is fixed.** All pin numbers, drivers, and capabilities are documented in `README.md`. Do not generalize — this is not a generic "ESP32-S3 wearable" project, it is the 2.06 watch project.

## Read Before Coding

1. `README.md` — full IC list, pin map, I²C addresses, roadmap. **The pins live there, not in your head.**
2. `D:\users\tj\Projects\SpawnWear\_vendor-waveshare-demo\` — cloned upstream Arduino + ESP-IDF demos (sits in the project parent folder, **outside git**). Read these before guessing how a chip works.
3. `D:\users\tj\Projects\SpawnWear\_wiki-decoded.txt` — decoded Waveshare wiki page for prose context (also outside git).
4. `D:\users\tj\Projects\NanoFrameTest1\NanoFrameTest1\` — the **reference architecture** for this project (different board, same shape: nanoFramework GATT + Blazor PWA + Playwright tests). When in doubt, mirror it.

## Architecture

Layered, Android-flavored. From bottom to top:

1. **HAL / Drivers** — hand-rolled C# (CO5300 QSPI, FT3168 I²C touch, AXP2101, PCF85063, QMI8658, ES8311, ES7210). Lowest layer, no framework dependencies above the runtime.
2. **System Services** — singletons started at boot: Power, WiFi, BLE, RTC, Audio, Storage, Sensors, Update, Logger. Apps consume services through interfaces; only one of each.
3. **UI Framework** — drawing primitives, framebuffer, touch + button dispatch, navigation stack, app lifecycle (`OnCreate` / `OnResume` / `OnPause` / `OnDestroy`), system widgets (status bar, keyboard, dialogs, list, slider, switch).
4. **Apps** — managed C# classes implementing the app lifecycle, run in-process under the SpawnWear app host. Built-ins: Launcher, Settings, Clock, AI Assistant (flagship), Media Player, Voice Recorder, Activity. Aim for OTA-installable apps once the foundation is solid.
5. **Companion Blazor WASM PWA** — talks BLE + WiFi to the watch, mirrors every built-in app as a remote-control surface. Uses `SpawnDev.BlazorJS` (no raw JS, no `IJSRuntime`).

### BLE GATT layout

- **Single `GattServiceProvider`** with one primary service. nanoFramework on ESP32 advertises one service at a time reliably.
- **Custom UUIDs** under base `a0e4f2c1-SSSS-CCCC-8000-00805f9b34fb` (note `c1` not `c0` — `c0` is NanoFrameTest1's namespace, this project uses `c1` so a phone with both PWAs installed doesn't get the device contracts confused).
- BLE GATT is just one of many transports the **BLE system service** owns. Apps don't talk to GATT directly — they talk to the service.
- BLE characteristics are user-toggleable via Settings → BLE; advertising is not "always on" the way it is in the demo scaffold. The demo scaffold (`WifiConfigService` etc) graduates from "always-on" to "user-controlled" once the Settings app exists.

## Rules

- **OS-shaped, not app-shaped.** Don't add features to `Program.cs` or `WifiConfigService.cs` directly. They're scaffolding. Real features go through a **system service** (a service singleton) and an **app** (a UI surface). If you can't decide which one a piece of functionality belongs in, write the question down and ask.
- **One owner per resource.** Display backlight, BLE radio, audio output, AXP2101 — each has exactly one system service that owns it. Apps ask through the service. Two apps grabbing the speaker simultaneously is a service-design failure, not an "apps fight it out" feature.
- **Lifecycle discipline.** Apps must implement `OnPause` / `OnResume` correctly. Anything an app starts (timer, BLE notify subscription, sensor stream, audio session) it must stop in `OnPause` and restart in `OnResume`. Background services keep ticking through pause / resume.
- **Power-aware by default.** Every service exposes a "low-power mode". The Power service (AXP2101 driver) coordinates: when the screen is off + idle, it tells everyone to drop to low-power, gates radios, dims rails. Don't write code that polls forever at full clock — use events, sleeps, and AXP2101 wake interrupts.
- **SpawnDev.BlazorJS for ALL JS interop.** No raw JavaScript. No `IJSRuntime`. (Companion PWA only.)
- **SpawnDev.RTC for WebRTC.** That's the AI Assistant transport. Don't write a parallel WebRTC stack.
- **Fix libraries, don't work around.** If `nanoFramework.Device.Bluetooth` / `nanoFramework.Hardware.Esp32` / SpawnDev.BlazorJS / SpawnDev.RTC is missing something, fix it upstream — see Rule 2 in `D:\users\tj\Projects\CLAUDE.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LostBeard/SpawnWear](https://github.com/LostBeard/SpawnWear) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->

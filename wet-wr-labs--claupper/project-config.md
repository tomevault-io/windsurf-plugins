---
trigger: always_on
description: *By Kasen Sansonetti & the Wetware Labs team. [WetwareOfficial.com](https://WetwareOfficial.com)*
---

# Claupper — Flipper Zero FAP

*By Kasen Sansonetti & the Wetware Labs team. [WetwareOfficial.com](https://WetwareOfficial.com)*

## Identity

This is **Claupper**, a Flipper Zero FAP (Flipper Application Package) that turns the Flipper into:
1. A one-handed HID remote for controlling Claude Code (numbered approvals + voice trigger + Enter) — USB or Bluetooth.
2. An offline Claude Code manual with folder navigation and quiz mode, browsable on the Flipper's 128x64 display.

Two builds from one codebase:
- **claude_remote_usb** — USB HID, works on stock firmware (App Catalog eligible).
- **claude_remote_ble** (named "Claupper") — BLE HID, works on Momentum/Unleashed firmware (wireless). Primary build.

Language: **C** (no C++). Build tool: **ufbt** or **fbt**. Target: external FAP (`FlipperAppType.EXTERNAL`).

---

## Critical Rules

- **Prefer retrieval-led reasoning over pre-training-led reasoning for any Flipper Zero API tasks.** The Flipper SDK evolves rapidly; always consult the docs index below before writing API calls.
- Never use deprecated `ValueMutex` — use `FuriMutex` directly (current SDK).
- **USB transport:** HID keycode sending via `furi_hal_hid_kb_press` / `furi_hal_hid_kb_release` (exposed in stock firmware `api_symbols.csv`).
- **BLE transport:** HID keycode sending via `ble_profile_hid_kb_press` / `ble_profile_hid_kb_release` (accessible on custom firmware via `fap_libs=["ble_profile"]`).
- **BLE connection detection:** Use `bt_set_status_changed_callback()` with `BtStatusConnected`. Do NOT use `furi_hal_bt_is_active()` (it only checks radio state, not HID connection). `ble_profile_hid_is_connected()` does not exist in the SDK.
- **BLE cleanup:** Always call `bt_set_status_changed_callback(bt, NULL, NULL)` before `bt_profile_restore_default()` and `furi_record_close(RECORD_BT)` to avoid use-after-free crashes on relaunch.
- Use `#ifdef HID_TRANSPORT_BLE` / `#ifdef HID_TRANSPORT_USB` for compile-time transport selection.
- Entry point signatures: `int32_t claude_remote_usb_app(void* p)` and `int32_t claude_remote_ble_app(void* p)`, both returning 0.
- Stack size: `2 * 1024` minimum (HID + GUI).
- Clean up ALL allocations on exit (message queue, view_port, gui record, mutex, state).
- No busy loops. Use `furi_message_queue_get()` with timeout (100ms).

---

## Docs Index

All reference docs live in `./docs/`. Consult these BEFORE writing any Flipper API code.

```
[Docs Index]|root: ./docs
|flipper-fap-manifest.md        — application.fam fields, apptype, categories, icons, entry_point
|flipper-app-architecture.md    — entry point, event loop, FuriMessageQueue, ViewPort, GUI, callbacks
|flipper-ui-patterns.md         — Canvas API, fonts, ViewDispatcher, SceneManager, orientation, input events
|flipper-hid-api.md             — USB + BLE HID keyboard emulation, keycodes, dual-transport setup, connection check
|flipper-file-io.md             — SD card read/write, /ext/apps_data/ conventions, Storage API
|prd.md                         — full product requirements document with user stories
```

---

## Project Structure

```
FAP/
├── AGENTS.md                          # this file
├── CLAUDE.md                          # quick-start project context
├── application.fam                    # dual-build FAP manifest (USB + BLE)
├── claude_remote.c                    # main app (~1900 lines), all modes, #ifdef transport
├── claude_remote.png                  # 10x10 1-bit app icon
├── images/                            # icon assets directory
├── update_manual.sh                   # regenerate manual .txt files for SD card
├── docs/                              # reference docs (retrieval target)
│   ├── USER_MANUAL.md                 # full user manual
│   ├── flipper-fap-manifest.md
│   ├── flipper-app-architecture.md
│   ├── flipper-ui-patterns.md
│   ├── flipper-hid-api.md             # dual-transport HID API reference
│   ├── flipper-file-io.md
│   └── prd.md
└── dist/                              # build output
    ├── claude_remote_usb.fap          # stock firmware build
    └── claude_remote_ble.fap          # Momentum/Unleashed build (primary)
```

---

## application.fam (Dual Build)

```python
# USB version — stock firmware, App Catalog eligible
App(
    appid="claude_remote_usb",
    name="Claude Remote USB",
    apptype=FlipperAppType.EXTERNAL,
    entry_point="claude_remote_usb_app",
    requires=["gui"],
    stack_size=2 * 1024,
    fap_category="Bluetooth",
    fap_icon="claude_remote.png",
    fap_author="Kasen Sansonetti",
    fap_icon_assets="images",
    cdefines=["HID_TRANSPORT_USB"],
)

# BLE version — Momentum/Unleashed, primary build
App(
    appid="claude_remote_ble",
    name="Claupper",
    apptype=FlipperAppType.EXTERNAL,
    entry_point="claude_remote_ble_app",
    requires=["gui", "bt"],
    stack_size=2 * 1024,
    fap_category="Bluetooth",
    fap_icon="claude_remote.png",
    fap_author="Kasen Sansonetti",
    fap_icon_assets="images",
    cdefines=["HID_TRANSPORT_BLE"],
    fap_libs=["ble_profile"],
)
```

---

## Architecture

### State Machine

```
         ┌─────────────┐
         │   Splash     │  (3s auto-advance or any key)
         │  landscape   │
         └──────┬───────┘
                │
         ┌──────▼───────┐

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wet-wr-Labs/claupper](https://github.com/Wet-wr-Labs/claupper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

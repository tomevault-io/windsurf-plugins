---
trigger: always_on
description: Enables automated and manual testing over serial (USB CDC). No LoRa radio is initialised — all mesh messages are simulated via injection. The radio hardware is never touched.
---

# SigurdOS T-Deck — Agent Onboarding

**You are an AI agent working on the SigurdOS T-Deck firmware.** This file is your instruction manual. Read it before modifying code.

**Do not modify this file or `CLAUDE.md` in any PR.** They are AI agent context. Only the repo owner changes them. Any PR that touches `AGENTS.md` or `CLAUDE.md` will be rejected without review.

---

## Markdown Files in This Repo

These are the reference documents you should load before starting work. Which one to read depends on what you're doing:

| File | When to read it | Purpose |
|------|----------------|---------|
| **`README.md`** | First time in the repo | Project overview, quick start, license |
| **`AGENTS.md`** ← you are here | Every session | Agent instructions, architecture, conventions, pitfalls |
| **`CLAUDE.md`** | Claude Code sessions | Same content — mirror of AGENTS.md |
| **`CONTRIBUTING.md`** | Before ANY task or PR | **Mandatory.** Full contribution workflow, issue-first requirement, PR checklist. AI agents must follow every step. |
| **`docs/KNOWN_ISSUES.md`** | Before feature work | What's broken or unfinished — don't duplicate effort |
| **`docs/CONTACT_STORE.md`** | When working on contact management | Contact store API, persistence, and data model |
| **`docs/LAUNCHER.md`** | When working on Launcher compatibility | Launcher detection, OTA gating, partition layout |
| **`docs/LOGGING.md`** | Before debugging serial output | Logging subsystem API, verbosity levels, and configuration |
| **`docs/MISSING_FEATURES.md`** | Before implementing new features | Catalog of MeshCore protocol features not yet implemented, with source references and effort estimates |
| **`firmware/README.md`** | Releasing or CI work | Release artifact structure, web flasher manifest format |
| **`test/README.md`** | Writing new tests | Test framework, mock structure, naming conventions |
| **`docs/HARDWARE.md`** | When working on hardware/drivers | Full hardware reference: pinout, boot sequence, peripheral details |
| **`docs/CHAT_SCREEN.md`** | When working on chat UI | Chat screen architecture, data model, input routing, persistence |
| **`docs/FEATURES_OVERVIEW.md`** | Getting oriented on features | Top-level index of all features with source cross-references |
| **`docs/HOME_SCREEN.md`** | When working on home screen | Home screen layout, tile grid, icon mapping |
| **`docs/ROADMAP.md`** | Understanding project direction | Development roadmap and planned features |
| **`docs/TERMINAL.md`** | When working on terminal | Terminal screen commands, serial protocol, debug levels |

**Critical rules — follow all:**
1. **Check for an existing issue on the upstream repo.** Before writing any code, check if there's already an open GitHub issue on `hermes-gadget/SigurdOS-tdeck` covering what you plan to do. If not, open one. No issue = no PR accepted.
2. **Read `docs/KNOWN_ISSUES.md`** before starting any feature work. If someone already tried and documented a problem, you'll find it there.
3. **Read `docs/MISSING_FEATURES.md`** before implementing any new feature. If the capability is listed there, don't duplicate the research — use the MeshCore source references and effort estimates provided.
4. **Follow `CONTRIBUTING.md`** — it's not optional. Every step applies to AI agents the same as human contributors.

---

## What Is This

Standalone T-Deck LVGL firmware that runs in the MeshCore mesh network. Think "Discord UI on a LoRa radio." Full mesh protocol compatibility — interoperates with any MeshCore node.

Three repos form the SigurdOS ecosystem:

| Repo | What | Stack |
|------|------|-------|
| `hermes-gadget/SigurdOS` | MeshCore fork (core library) | C++/PlatformIO, ESP32 |
| **`hermes-gadget/SigurdOS-tdeck`** ← **you are here** | T-Deck LVGL firmware | C++/PlatformIO, LVGL v9, LovyanGFX |
| `hermes-gadget/SigurdOS-client` | Flutter mobile app | Dart/Flutter, BLE/USB/TCP |

---

## Quick Start

```bash
git clone --recurse-submodules git@github.com:hermes-gadget/SigurdOS-tdeck.git
cd SigurdOS-tdeck

# Run all native tests (no hardware, fast — do this repeatedly)
pio test -e native_test -v

# Run one test module
pio test -e native_test -f test_keyboard -v

# Build firmware
pio run -e SigurdOS_TDeck

# Check test count (varies as tests are added)
pio test -e native_test --list-tests
```

MeshCore is at `lib/meshcore/` — a git submodule. `git submodule update --init` if you cloned without `--recurse-submodules`.

---

## Architecture

```
src/
├── main.cpp               # Boot: board→display→SPIFFS→splash→mesh→UI (deferred input init)
├── lv_conf.h              # LVGL v9 config (16-bit, partial render)
├── hal/
│   ├── tdeck_pins.h       # Full T-Deck pinout + SPI/I2C aliases + SIGURDOS_VERSION
│   ├── tdeck_board.h      # MainBoard impl (sleep, battery, power)
│   ├── display.cpp/h      # LovyanGFX ST7789 + LVGL + touch/keyboard callbacks
│   ├── battery.cpp/h      # ADC mV→%
│   ├── touch.cpp/h        # GT911 capacitive touch (I2C, 400 kHz)
│   ├── keyboard.cpp/h     # I2C keyboard (ESP32-C3 MCU at 0x55)
│   ├── trackball.cpp/h    # 5-direction trackball (debounce, event queue)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hermes-gadget/SigurdOS-tdeck](https://github.com/hermes-gadget/SigurdOS-tdeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->

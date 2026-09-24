---
trigger: always_on
description: <a href="AGENTS.zh_CN.md">简体中文</a> · <strong>English</strong>
---

<p align="right">
  <a href="AGENTS.zh_CN.md">简体中文</a> · <strong>English</strong>
</p>

# Repository Guidelines for AI Agents

This file is the only mandatory entry point for AI-assisted work in this repository. Read task-specific documents from the routing table below; do not load every README by default.

## Required AI skills

The following five skills are required for AI-assisted development in this repository:
`passport-develop`, `passport-setup`, `passport-build`, `passport-device-test`, and
`passport-debug`. Their maintained sources are under `skills/`.

Before starting development, the AI must check that all five are installed and
available in its current environment. If any are missing, the AI must choose an
appropriate installation method for its tool and environment, perform the
installation, and verify availability itself. Do not wait for the user to request
installation or make the user choose the method or run installation commands.
No particular script, installation directory, or AI client is mandatory.

Respect the environment's approval requirements and preserve existing skills
and user configuration. If permissions, conflicts, or platform limitations prevent
installation, explain the blocker and request only the necessary user action;
do not claim installation succeeded. Having all five available does not mean
loading or invoking all five for every task: use only the matching skills, and
retain the separate authorization requirements for flashing, Git writes, and publishing.

## Project and safety baseline

- Target: ESP32-C3, 8 MB Flash, no PSRAM, ESP-IDF 5.5.3.
- Keep the repository's default partition table minimal: NVS, PHY data, and
  one factory application spanning the rest of the 8 MB Flash. User firmware
  may deliberately change this layout; validate the resulting table and do not
  turn product-specific partitions into mandatory template contracts.
- Preserve existing user changes. Start with `git status --short --branch`; never overwrite or clean unrelated files.
- Flashing new firmware does not require backing up the firmware already on the device; do not make a Flash readback a prerequisite. This does not guarantee preservation of user data or authorize a full-chip erase. Follow the [flashing and data policy](docs/development/engineering/firmware-layout.md#flashing-and-stored-data).
- Hardware facts follow this priority: product specifications and measured results → `components/bsp/include/bsp_pins.h` → BSP headers and implementation → hardware guide → README/demo code. If a task requires a hardware detail not defined by these sources, ask the user instead of guessing.
- Reusable board logic belongs in `components/bsp`; pages, state machines, animations, and application tasks belong in `main`.
- Derivative applications must redesign and implement their own UI. Reusing the current demo test menu, screens, or visual shell is prohibited; renaming, recoloring, or adding a feature to that shell does not count as a redesign. BSP APIs and non-UI logic remain reusable. See the [mandatory UI redesign rule](docs/development/ai-guide.md#mandatory-ui-redesign-for-derivative-applications).
- LVGL is not thread-safe. Code outside the LVGL task must hold `bsp_lvgl_lock()` while accessing LVGL objects.
- Before adding Chinese UI text, follow the [font checklist](docs/development/engineering/coding-conventions.md#chinese-fonts-and-missing-glyphs). The default Montserrat fonts have no Chinese glyphs; UTF-8 and a successful build do not prove display support. Verify glyph coverage, widget font selection, and on-device rendering.
- Button callbacks must stay non-blocking. Audio, storage, networking, and other slow operations belong in worker tasks.
- A demo must stop every task, timer, callback, and event handler that can access its UI before deleting the screen.
- Keep testable state machines, protocols, timing, and layout calculations independent from ESP-IDF/LVGL and cover them with host tests.
- Never commit credentials, device QR secrets, private keys, personal data, or unsanitized logs.
- Every maintained Markdown document uses English at its default `.md` path and Simplified Chinese in a paired `.zh_CN.md` file. Keep both versions aligned and retain reciprocal language links.

## Task-specific context routing

| Task | Read before editing |
| --- | --- |
| Any code change | `docs/development/ai-guide.md`, relevant headers and neighboring implementation |
| Application workflow or core skill setup | `skills/README.md`; ensure the five required skills are available, then use only the matching skill |
| Environment bootstrap or missing toolchain | `docs/development/engineering/environment-setup.md` |
| BSP, pins, buses, display, audio, battery | `docs/hardware-design/AI_HARDWARE_DEVELOPMENT_GUIDE.md`, `components/bsp/include/bsp_pins.h` |
| Demo or menu | `main/demo.h`, `main/main.c`, the nearest `main/demo_*.c` implementation |
| Chinese UI text or fonts | `docs/development/engineering/lvgl-chinese-fonts.md`, the application's font assets, configuration, and widget styles |
| Wi-Fi connection or Bluetooth provisioning | `docs/development/engineering/wifi-provisioning.md`, the referenced `demo/blufi-provisioning` implementation |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FoloToy/ai-passport](https://github.com/FoloToy/ai-passport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

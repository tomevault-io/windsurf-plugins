---
trigger: always_on
description: This app is a Vue 3 + TypeScript + Vite pin explorer for Espressif SoCs and development boards. It uses Vuetify 4 for UI, Pinia for state, and data-driven SVG components for clickable chip packages and board headers.
---

# ESP Pinout Explorer Agent Notes

This app is a Vue 3 + TypeScript + Vite pin explorer for Espressif SoCs and development boards. It uses Vuetify 4 for UI, Pinia for state, and data-driven SVG components for clickable chip packages and board headers.

## Core Principle

Accuracy matters more than UI flourish. Raw SoC package data must come from official Espressif datasheets. Board profiles must come from official Espressif board user guides, official schematics, or official board documentation. Do not infer pinout data from random tables, blogs, product listings, or screenshots.

## Current Scope

- Implemented SoCs:
  - ESP32-S3 QFN56
  - ESP32-C6 QFN40
  - ESP32-C6 QFN32
- Implemented board profiles:
  - ESP32-S3-DevKitC-1 v1.1
  - ESP32-S3-DevKitM-1
  - ESP32-S3-USB-OTG
- Future work is tracked in `todo.md`.
- The README documents install, dev, build, and top-level structure.

## Important Files

- `src/data/socs/esp32s3.ts`: ESP32-S3 pin metadata.
- `src/data/socs/esp32c6.ts`: ESP32-C6 pin metadata and package variants.
- `src/types/soc.ts`: shared SoC and pin types.
- `src/stores/socStore.ts`: selected SoC, selected package, selected pin, and search/filter state.
- `src/components/AppShell.vue`: compact app bar, desktop sidebar layout, mobile control drawer, and main app frame.
- `src/components/ExplorerSidebar.vue`: SoC/profile selectors, selected SoC/profile chips, search, pin count, and legend.
- `src/components/ChipSvg.vue`: data-driven clickable SVG chip drawing.
- `src/components/BoardSvg.vue`: data-driven clickable SVG development-board header drawing.
- `src/components/SocPinoutView.vue`: focused pinout stage and right-side selected-pin drawer.
- `src/components/PinInfoDrawer.vue`: selected-pin details, function chips, warnings, notes, and source link.
- `src/components/InfoTooltip.vue`: reusable info icon popover for technical section headings.
- `src/components/FunctionChip.vue`: reusable function chip with optional popover description.
- `src/data/functionDescriptions.ts`: dictionary and pattern matcher for explaining non-obvious function names.
- `src/data/pinWarnings.ts`: presentation rules that split official warning categories into maker warnings and board design notes.

## Pin Data Rules

- Preserve the `SocDefinition`/`SocPin` shape from `src/types/soc.ts`.
- Add one data file per SoC family unless the package divergence is large enough to justify separation.
- Use `packageVariants` when a SoC has multiple package pinouts.
- Keep `source` metadata current: title, datasheet version, URL, and relevant sections.
- Every package pin should have a stable `id`, package `number`, `name`, `type`, `position`, and `mainFunctions`.
- Set `gpio` only when the package pin exposes a GPIO number.
- Add `warnings` for official cautions such as strapping, boot, USB, flash, PSRAM, JTAG, UART0, reset, voltage, power, glitch, or known boot restrictions.
- Do not omit official warnings just because they are low priority for makers. `src/data/pinWarnings.ts` decides which warnings get yellow maker-warning treatment versus calmer board-design-note treatment.
- Add `keywords` for search terms that users reasonably expect, such as `boot`, `strap`, `adc`, `touch`, `usb`, `spi`, `uart`, `jtag`, `flash`, `psram`, and package-specific function aliases.

## Board Profile Rules

- Use `boardProfiles` on the related `SocDefinition`.
- Set `kind: 'board'`, a stable `id`, a concise `name`, and a maker-facing `packageName`.
- Add `moduleNames` and `identificationNotes` when the visible module marking differs from the dev-board profile name.
- Use `boardLayout: 'connector-groups'` and `boardGroup` for official board pin tables that are grouped by function or connector instead of J1/J3-style side headers.
- Use official board header identifiers in `displayNumber`, for example `J1-4`.
- Preserve the board silkscreen/header label in `boardLabel`, for example `TX`, `3V3`, or `14`.
- Preserve official numeric GPIO-style board labels exactly in `boardLabel`, for example `IO23` or `GPIO23`. `BoardSvg.vue` normalizes these to the short visible label `23` so board drawings stay consistent across profiles, while the drawer and search still expose the official board label.
- Set `boardHeader` to the official header block name, for example `J1` or `J3`.
- For GPIO header pins, copy or derive SoC-level GPIO metadata from the raw SoC package pin so drawer sections, search, warnings, and tooltips stay consistent.
- Add board-specific maker warnings for pins connected to on-board hardware, boot/reset buttons, USB, UART bridges, LEDs, or module memory constraints.
- If a board exposes pins used internally for flash or PSRAM, keep the official flash/PSRAM warning and add a maker-visible board warning such as `onboard` or `psram` so the SVG badge and Safe use filter treat the pin as risky.
- Keep board-only power and ground pins as real clickable pins. They should have clear notes and search keywords even when there is no GPIO.
- Board profiles should use `BoardSvg.vue`; raw package profiles should use `ChipSvg.vue`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thelastoutpostworkshop/esp-pinout-explorer](https://github.com/thelastoutpostworkshop/esp-pinout-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->

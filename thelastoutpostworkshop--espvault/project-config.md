---
trigger: always_on
description: ESP Board Vault is a free, local-first desktop application for ESP32 makers.
---

# AGENTS.md - ESP Board Vault

## Project Overview

ESP Board Vault is a free, local-first desktop application for ESP32 makers.
It is a smart local notebook for remembering ESP32 boards, firmware, projects,
hardware capabilities, notes, and physical locations.

The first version is a standalone Electron desktop app built from a Vue 3 web app.
There is no hosted backend, no user accounts, no cloud sync, no payment system,
and no telemetry.

## Working Preferences

Always include a short commit comment suggestion in the final response.

For browser-based visual checks of the Vue renderer, use local Playwright
against the browser harness. Do not rely on the Codex in-app browser backend
for this repo's visual testing path.

```bash
npm run test:visual
```

Use `npm run test:visual:headed` when interactive inspection is useful. For
manual exploratory checks, run `npm run dev:browser` and open the printed
`browser-harness.html` URL. The harness provides a typed mock preload API and
seeds sample boards and projects in browser IndexedDB when empty, which lets
renderer pages be inspected without Electron.

## UI Style Guidelines

Keep the interface colorful, modern, and professional for ESP32 makers. The app
should feel like a focused electronics workbench: technical, clean, and useful,
without looking childish or like a marketing landing page.

Use the existing Vuetify theme system and shared renderer styles before adding
one-off visual rules. Preserve both light and dark mode support when changing
colors, surfaces, borders, shadows, chips, tables, cards, or empty states. Avoid
hard-coded light-only colors in page-scoped styles; prefer Vuetify theme tokens
and shared CSS variables from `src/renderer/styles.css`.

Use consistent 8px rounded corners, subtle shadows, clear section spacing, and
theme-aware borders. Cards should be used for real panels, repeated items, and
tools, not as nested decoration. Keep operational pages dense enough for repeat
use while still readable.

Use icons for navigation, actions, tool cards, status chips, and empty states.
Status chips should remain color-coded and easy to scan. Empty states should
explain the next useful action and include a clear call to action when one is
available.

Prefer list/detail layouts for inventory management pages where records need a
selected detail view. The Projects and Boards pages establish the current
pattern: a selectable list on the left and a detailed record view on the right,
collapsing responsively on smaller screens.

Tools and external resources should be presented as curated in-app pages when
they need descriptions or context. External links must continue to open through
the typed preload API, not direct renderer Node.js or raw IPC.

## Technical Stack

Use Electron, Vue 3, TypeScript, Vite, Vuetify 4, Pinia, Dexie, and
IndexedDB. Use `tasmota-webserial-esptool` for ESP board scanning through Web
Serial.
https://github.com/Jason2866/WebSerial_ESPTool/tree/development

Use Dexie from the Vue renderer for structured local data. The renderer must
not use Node.js APIs directly. It communicates with the main process only for
privileged operations through a typed preload API exposed with `contextBridge`.

## Architecture

```text
Vue Renderer
  -> Pinia stores
  -> Repository interfaces
  -> Storage implementation
     -> Dexie / IndexedDB for structured local data

Vue Renderer
  -> Preload API using contextBridge
  -> Electron IPC only for privileged operations
  -> Main Process Services for serial, files, export/import, and attachments
```

Structured local data belongs in the renderer storage layer. Electron main and
preload must not own board CRUD or other normal inventory CRUD unless the data
operation needs privileged OS access.

## Storage Abstraction

Keep storage replaceable. Vue pages and components must not import Dexie,
IndexedDB helpers, or concrete storage implementation files directly.

Use this dependency direction:

```text
Vue pages/components
  -> Pinia stores
  -> repository interfaces from src/renderer/repositories/
  -> implementation selected in src/renderer/repositories/index.ts
  -> concrete storage under src/renderer/storage/{provider}/
```

Current implementation:

```text
src/renderer/repositories/BoardRepository.ts
src/renderer/repositories/ProjectRepository.ts
src/renderer/repositories/BackupRepository.ts
src/renderer/repositories/index.ts
src/renderer/storage/dexie/DexieBoardRepository.ts
src/renderer/storage/dexie/DexieProjectRepository.ts
src/renderer/storage/dexie/DexieBackupRepository.ts
src/renderer/storage/dexie/vaultDatabase.ts
```

When adding new data areas such as projects, firmware history, attachments
metadata, pin assignments, or settings:

1. Define a storage-neutral repository interface in `src/renderer/repositories/`.
2. Implement it under `src/renderer/storage/dexie/`.
3. Wire it through `src/renderer/repositories/index.ts`.
4. Use the repository from Pinia stores.

To switch storage in the future, add a new implementation folder such as:

```text
src/renderer/storage/file/
src/renderer/storage/sqlite/
```

Then change only the implementation wiring in
`src/renderer/repositories/index.ts` where practical.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thelastoutpostworkshop/ESPVault](https://github.com/thelastoutpostworkshop/ESPVault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->

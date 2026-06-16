---
trigger: always_on
description: This document provides a comprehensive guide for developers and AI agents working on the `tab-positioner` Chrome extension project. It outlines the project structure, technology stack, and development workflows.
---

# Developer/AI Agent Instructions for Tab Positioner

This document provides a comprehensive guide for developers and AI agents working on the `tab-positioner` Chrome extension project. It outlines the project structure, technology stack, and development workflows.

## 1. Project Overview

`tab-positioner` is a Google Chrome extension designed to transform tab management behavior. It is a Manifest V3 extension.

## 2. Technology Stack

- **Runtime**: Node.js
- **Package Manager**: `pnpm` (implied by `pnpm-lock.yaml`)
- **Languages**:
  - **TypeScript**: Primary language for logic (`.ts`, `.tsx`).
  - **SCSS**: For styling (`.scss`).
- **Build System**:
  - **Eleventy**: Orchestrates the build process and handles HTML generation.
  - **Vite**: Used as a plugin within Eleventy to bundle JavaScript and CSS.
- **Testing**:
  - **Playwright**: For End-to-End (E2E) testing.
- **UI Framework**:
  - Uses `jsx-async-runtime`, a lightweight JSX implementation for the Options/Popup UI. The codebase manually handles form state and restoration without a heavy framework like React.

## 3. Project Structure

The project is organized as follows:

- **`root`**:
  - `manifest.json`: The Manifest V3 configuration. **Source of Truth** for extension metadata, permissions, and entry points.
  - `eleventy.config.mjs`: Main build configuration file. Defines how `src` files are processed into `dist`.
  - `package.json`: Node.js project configuration. Lists dependencies, scripts, and metadata.
  - `tsconfig.json`: TypeScript configuration.
  - `TESTING.md`: Manual testing checklist.
- **`src/`**: Source code directory.
  - **`background/`**: Contains the code for the extension's Service Worker (Background Script).
    - `main.ts`: Entry point. Initializes `Listeners`, `SyncSettings`, and `TabsInfo`.
    - `syncsettings.ts`: Synchronizes settings from `chrome.storage` into memory/session for fast access.
    - `tabsinfo.ts`: Maintains state about tabs to support decision making (e.g. creation delay, recent tabs).
    - `tabmover.ts`: Core logic for moving normal and popup tabs (`tabMover`).
    - `tabcreation.ts`: Core logic for where to place new tabs (`createdTabMover`).
    - `tabactivation.ts`: Core logic for which tab to activate after closing one (`tabRemovedActivater`).
    - `popupcreation.ts`: Core logic for handling popup tab creation (`createdPopupMover`).
  - **`options/`**: Contains code for the Extension Options Page and Popup.
    - `index.tsx`: Main UI entry point (JSX).
    - `options.ts`: Logic for loading/saving settings and form interactions.
    - `settings.tsx`: Dynamically generates UI components based on the shared setting schemas.
    - `options.scss`: Stylesheets.
  - **`shared/`**: Shared utilities and constants used by both background and options contexts.
    - `constants.ts`: Global constants (e.g., delays, magic URIs).
    - `i18n.ts`: Type-safe i18n helpers. Imports English messages for compile-time key validation.
    - `listeners.ts`: Wrapper for Chrome events. It forces async listener callbacks to execute synchronously and sequentially.
    - `session.ts`: Base class for synchronizing singleton state with `chrome.storage.session`.
    - `settings.ts`: **Critical File**. Defines `ExtensionSettings`, `SETTING_SCHEMAS`, and default values.
    - `storage.ts`: Handles reading/writing to `chrome.storage`.
    - `logging.ts`: Centralized logging with configurable log levels.
  - **`types/`**: TypeScript type definitions.
- **`scripts/`**: Build and maintenance scripts.
  - `clean.ts`: Cleans the `dist/` directory.
  - `locales.ts`: Validates i18n locale files against the English base.
  - `package.ts`: Packages the extension into a `.zip` for the Chrome Web Store.
  - `package.sh`: Bash alternative to `package.ts` (NOT preferred).
- **`tests/`**: Automated Playwright tests.
  - **`ext/`**: A helper extension used in tests to access `chrome` APIs.
  - `fixtures.ts`: Test fixtures.
  - `helpers.ts`: Test helper functions.
  - `constants.ts`: Test constants (e.g., delays, URIs).
  - `extension.spec.ts`: Basic extension loading test.
  - `scenarios/*.spec.ts`: Test scenarios organized by feature.
- **`utils/`**: Utilities used by the **build process** (Eleventy plugins, git info, etc.).
  - `comments.ts`
  - `gitinfo.ts`
  - `manifest.ts`
  - `vite.ts`
  - `tsx.ts`
- **`_locales/`**: i18n message configurations (e.g., `en/messages.json`).

## 4. Codebase Deep Dive

### Shared Logic & Data Models

- **Settings Schema**: Defined in `src/shared/settings.ts`.
  - `ExtensionSettings`: The interface for all setting keys and values.
  - `SETTING_SCHEMAS`: Metadata for each setting (type, potential choices, i18n keys). This drives the UI generation in `src/options/settings.tsx`.
  - **Advanced Settings**: Keys starting with an underscore (e.g., `_debug_mode`) are functionally treated as **Advanced Settings**. The UI generation logic (`src/options/settings.tsx`) uses this prefix to automatically filter and place them in the "Advanced" details section.

### Background Service Worker

- **Initialization** (`src/background/main.ts`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huangyxi/tab-positioner](https://github.com/huangyxi/tab-positioner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

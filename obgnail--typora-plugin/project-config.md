---
trigger: always_on
description: This file provides guidance to AI coding assistants and agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants and agents when working with code in this repository.

## Project Overview

Typora Plugin is an extensible plugin system for the Typora Markdown editor. It injects into Typora's Electron-based runtime (via `window.html`) and provides 50+ plugins. The project is pure JavaScript (no TypeScript), requires Typora >= 0.9.98, and supports Windows and Linux.

**Compatibility Target:**

Because this project must support Typora 0.9.98, any code injected into the Typora runtime must be compatible with its underlying legacy Electron version. The minimum supported environment limits are:

- **Chrome**: 83.0.4103.122
- **Node.js**: 12.14.1

## Commands

All commands run from the `develop/` directory. Requires Node.js >= 22.

```bash
cd develop
npm install          # Install dev dependencies

# Testing (uses Node.js built-in test runner: node:test + node:assert)
npm test                                    # Run all tests
node --require ../plugin/global/core/polyfill.js --test test/utils.test.js   # Run single test file

# Building vendored dependencies (esbuild bundles NPM packages into plugin/global/core/lib/)
npm run build:all                           # Build all vendors
npm run build:single                        # Build a single vendor (edit arg in package.json, e.g. "katex")
npm run build:download                      # Build download-type vendors (js-yaml, markdown-it, etc.)

# Development (requires TYPORA_PATH set in develop/.env)
npm run dev                                 # Development mode
npm run sync                                # Watch plugin/ for changes, sync to Typora install dir
npm run serve                               # Sync + auto-restart Typora via JSON-RPC
npm run rpc                                 # JSON-RPC connection to running Typora
```

## Architecture

### Entry Point Flow

1. `plugin/index.js` -- loaded by modified `window.html`, requires `plugin/global/core/index.js`
2. `plugin/global/core/index.js` -- `entry()` function: checks Typora version, reads TOML settings, sets up globals (`BasePlugin`, `BaseCustomPlugin`), initializes i18n, loads all plugins via mixin chain, publishes `allPluginsHadInjected` event

### Core Framework (`plugin/global/core/`)

- **`plugin.js`** -- Defines `IPlugin` (base interface), `BasePlugin`, `BaseCustomPlugin` classes, and `LoadPlugins()` which drives the plugin lifecycle
- **`serviceContainer.js`** -- Singleton storing all plugin instances and settings; provides lookup APIs (`getBasePlugin()`, `tryGetPlugin()`)
- **`i18n.js`** -- i18n system supporting `en`, `zh-CN`, `zh-TW`; loads JSON locale files from `plugin/global/locales/`
- **`polyfill.js`** -- Polyfills for older Electron/Node (`Object.hasOwn`, `Promise.withResolvers`, etc.)

### Core Utilities (`plugin/global/core/utils/`)

- **`index.js`** -- Large utility class (~200+ static methods): DOM manipulation, file ops, path handling, HTML/CSS injection. Instantiates all mixins.
- **`eventHub.js`** -- Event bus with typed events (`fileOpened`, `fileEdited`, `outlineUpdated`, etc.). Uses MutationObserver and decorator hooks.
- **`hotkeyHub.js`** -- Global hotkey registration/dispatch. Normalizes key combos (Ctrl+Shift+Alt+Key).
- **`decorator.js`** -- AOP system. Wraps Typora's internal functions with before/after hooks, argument/result modification, call prevention. Supports decorator chaining with priorities.
- **`settings.js`** -- TOML settings reader (default + user), supports save/import/export, auto-save via Proxy.
- **`styleManager.js`** -- CSS loading with template variable substitution (`${config.value}`).
- **`thirdPartyDiagramParser.js`** -- Extended diagram framework with lazy-loading and export support.

### Plugin System

**Plugin Lifecycle** (in order): `prepare()` -> `style()` -> `html()` -> `hotkey()` -> `init()` -> `process()` -> `finalize()`

**Two plugin types:**

1. **Base Plugins** (in `plugin/`) -- Extend `BasePlugin`, implement `call(action, meta)`. Can be single-file (`plugin/{name}.js`) or directory-based (`plugin/{name}/index.js` with resources).
2. **Custom Plugins** (in `plugin/custom/plugins/`) -- Extend `BaseCustomPlugin`, implement `selector()`, `hint()`, `callback()`. Managed by the `custom` base plugin (`plugin/custom/index.js`). User-populated, empty by default.

### Settings System (`plugin/global/settings/`)

- `settings.default.toml` (~120KB) -- Default config for all base plugins. Each plugin has `[plugin_name]` section with `ENABLE`, `NAME`, and plugin-specific options.
- `settings.user.toml` -- User overrides
- `custom_plugin.default.toml` / `custom_plugin.user.toml` -- Same pattern for custom plugins
- Supports home directory override (`~/.config/typora_plugin/`) for persistence across updates

### Key Patterns

- **Service Container / DI**: `ServiceContainer` singleton holds all plugin instances, accessible via `utils.container`
- **AOP (Aspect-Oriented Programming)**: `decorator.js` wraps Typora internals without modifying source. Used by `eventHub`, `exportHelper`, and many plugins.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obgnail/typora_plugin](https://github.com/obgnail/typora_plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

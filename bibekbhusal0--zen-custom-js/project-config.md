---
trigger: always_on
description: This document provides context for code assistants working on this Zen Browser customization repository.
---

# Code Assistant Context

This document provides context for code assistants working on this Zen Browser customization repository.

## Project Overview

Collection of user scripts and CSS modifications for Zen Browser (Firefox-based). Each feature is self-contained in its own directory. Uses Bun as the build system to bundle scripts into `.uc.js` files.

**Key Technologies**: JavaScript (ESM), CSS, Bun.

## Build Commands

```bash
# Build all mods
bun run build

# Build specific mod
bun run build:browsebot
bun run build:palette
bun run build:reopen
bun run build:sidebar
bun run build:select
bun run build:search

# Development mode with watch (auto-rebuild on changes)
bun run dev                    # Only watches browse-bot
bun run dev:browsebot          # Watch specific mod
bun run dev:palette
bun run dev:reopen
bun run dev:sidebar
bun run dev:select
bun run dev:search

# Other commands
bun run format                 # Format code with Prettier
bun run lint                   # Run ESLint
```

**No test framework exists** - manual testing in browser required. Bundled files go to `dist/`.

## Code Style Guidelines

### Imports and Modules

- Use ES6 modules (`import`/`export`)
- Import Firefox globals via `ChromeUtils.importESModule`
- Shared utilities from `utils/` directory
- Relative imports: `../utils/` for shared, `./utils/` for mod-specific

### Formatting (Prettier)

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "semi": true,
  "singleQuote": false,
  "trailingComma": "es5",
  "arrowParens": "always",
  "bracketSameLine": true,
  "bracketSpacing": true,
  "quoteProps": "as-needed"
}
```

Run `npm run format` before committing. Formatting is also automated on push.

### Naming Conventions

- **Variables/functions**: `camelCase` (`getPreference`, `userConfig`)
- **Classes**: `PascalCase` (`ZenCommandProvider`)
- **Constants**: `UPPER_SNAKE_CASE` (`MAX_RECENT_COMMANDS`, `PREFS`)
- **Private methods**: underscore prefix (`_closeUrlBar`, `_validateInput`)
- **Event handlers**: descriptive (`handleUrlbarClose`, `onUrlbarClose`)
- **DOM elements**: descriptive (`inputField`, `resultElement`)

### Types and Documentation

No TypeScript - use JSDoc comments:

```javascript
/**
 * Executes a command by its key.
 * @param {string} key - The command identifier.
 * @returns {Promise<void>}
 */
async executeCommandByKey(key) { ... }
```

### Error Handling

Wrap potentially failing code in try-catch. Use `debugError(...)` for logging. Return fallback values on errors.

```javascript
try {
  const pref = getPref(key);
  if (!pref.exists()) return defaultValues[key];
  return pref.value;
} catch (e) {
  debugError("Error getting preference:", e);
  return defaultValues[key];
}
```

For simpler operations, the utility functions handle errors internally:

- `getPref(key, defaultValue)` returns defaultValue on error
- `setPref(key, value)` silently fails if pref doesn't exist
- `resetPref(key)` checks if pref exists before clearing

### Debug Logging

- `debugLog(...)` for debug output (only in debug mode)
- `debugError(...)` for errors
- Prefix logs with mod name: `console.log("Palette:", ...args)`
- Debug mode controlled by preference key (`PREFS.DEBUG_MODE`)

### Firefox Integration

- Use `ChromeUtils.importESModule()` for Firefox modules
- Use utility functions from `utils/` for prefs, widgets, and other browser integration
- Pre-defined globals (eslint.config.js): `gBrowser`, `Services`, `ChromeUtils`, etc.

### Preferences Pattern

Use the base `PREFS` class from `utils/pref.js` and extend it for each mod:

```javascript
import { PREFS as BasePREFS, addPrefListener, removePrefListener } from "../../utils/pref.js";

class ModNamePREFS extends BasePREFS {
  static MOD_NAME = "ModName";
  static DEBUG_MODE = "mod-name.debug-mode";
  static ENABLED = "mod-name.enabled";

  static defaultValues = {
    [ModNamePREFS.DEBUG_MODE]: false,
    [ModNamePREFS.ENABLED]: true,
  };

  static get enabled() {
    return this.getPref(this.ENABLED);
  }

  static set enabled(value) {
    this.setPref(this.ENABLED, value);
  }
}

export const PREFS = ModNamePREFS;
```

The base class provides:

- `getPref(key, defaultValue)` - Get a preference value
- `setPref(key, value)` - Set a preference value
- `setInitialPrefs()` - Initialize default preferences
- `debugMode` getter/setter - Access debug mode preference
- `debugLog(...args)` - Log debug messages (only when debugMode is true)
- `debugError(...args)` - Log error messages (only when debugMode is true)

For preference listeners, use the standalone functions:

```javascript
// Add a listener - returns an object with name and callback
const listener = addPrefListener(PREFS.ENABLED, (pref) => {
  console.log("Enabled changed:", pref.value);
});

// Remove the listener later
removePrefListener(listener);
```

### DOM Manipulation

Use `parseElement()` from `utils/parse.js`:

```javascript
import { parseElement } from "../utils/parse.js";
const element = parseElement(`<div class="my-class">Content</div>`);
```

For XUL elements (Firefox UI), specify type:

```javascript
const xulElement = parseElement(`<toolbarbutton id="my-btn"/>`, "xul");
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BibekBhusal0/zen-custom-js](https://github.com/BibekBhusal0/zen-custom-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

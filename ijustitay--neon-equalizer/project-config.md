---
trigger: always_on
description: Everything an AI agent needs to work on this codebase safely without breaking things.
---

# Neon Equalizer — AI Agent Context

Everything an AI agent needs to work on this codebase safely without breaking things.

---

## What This App Is

**Neon Equalizer** is a Windows desktop GUI for [Equalizer APO](https://sourceforge.net/projects/equalizerapo/). It is an Electron app (no React, no Vue — vanilla JavaScript + Vite).

Users use it to:
- Edit parametric/graphic EQ filters and save them to Equalizer APO's `config.txt`
- Load AutoEQ headphone targets and generate matching filters automatically
- Browse Squiglink measurements and headphone frequency responses
- Use VST plugins, convolution, loudness correction, hardware PEQ transfer

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Desktop shell | Electron 41 |
| Bundler | Vite 6 |
| UI | Vanilla JavaScript (no framework) |
| Styling | Plain CSS with custom properties |
| Build output | Windows `.exe` (portable + NSIS installer) |
| Package manager | npm |

**There is no React, Vue, Angular, or TypeScript.** Do not introduce them.

---

## Project Structure

```
electron/
  main.js          Main process — window creation, IPC handlers, file I/O, APO detection, auto-updater
  preload.js       Context bridge — exposes apoAPI and windowAPI to renderer

src/
  main.js          Renderer entry point — ALL UI logic (~4600 lines)
  index.css        Design system + all component styles (~2900 lines)
  components/
    frequencyGraph.js    Canvas-based FR graph — drag bands, zoom, pan, curves (~1700 lines)
    parametricEQ.js      Parametric filter table — renders rows, handles input
    graphicEQ.js         Graphic EQ sliders
    autoEQEngine.js      AutoEQ optimizer — generates PEQ filters from target + measurement
    audioPlayer.js       Live EQ preview player (pink noise, white noise, file)
    targetLoader.js      Loads and parses EQ target curves
    targetAdjustments.js Bass/treble/tilt sliders applied before AutoEQ
    squiglink.js         Squiglink embedded browser panel
    squiglinkDB.js       Squiglink headphone database search
    hidDevice.js         Hardware PEQ USB/network transfer
    devicePeq/           Device-specific PEQ protocol handlers

  config/
    parser.js      Parses Equalizer APO config.txt text → JS object
    serializer.js  Serializes JS config object → Equalizer APO config.txt text

public/
  targets/         Bundled EQ target curve files (.txt)

assets/
  icon.svg         Logo source (edit this to change the icon)
  icon.ico/.png    Generated — do NOT edit manually, run `pnpm run icons`

docs/
  images/          README screenshots
  USAGE.md         Usage guide

scripts/
  generate-icons.mjs    Generates icon assets from icon.svg using sharp
  take-screenshots.ps1  Helper to auto-capture screenshots of the running app
```

---

## IPC Bridge (renderer ↔ main process)

The renderer calls main-process functions via two bridges exposed in `electron/preload.js`:

```javascript
window.apoAPI.readConfig(filePath)        // Read a file from disk
window.apoAPI.writeConfig(filePath, text) // Write a file to disk
window.apoAPI.getAPOPath()                // Detect Equalizer APO config dir
window.apoAPI.selectFile(options)         // Open file picker dialog
window.apoAPI.saveFile(content, options)  // Save file dialog
window.apoAPI.selectConfigDir()           // Pick config folder
window.apoAPI.listConfigFiles(dir)        // List files in a directory
window.apoAPI.fetchText(url, options)     // HTTP fetch (bypasses CORS)
window.apoAPI.listAudioDevices()          // List audio output devices
window.apoAPI.captureRegionImage(rect)    // Screenshot a screen region
window.apoAPI.openExternalUrl(url)        // Open URL in browser
window.apoAPI.getUpdaterState()           // Auto-updater state
window.apoAPI.checkForUpdates()
window.apoAPI.downloadUpdate()
window.apoAPI.installUpdate()
window.apoAPI.onUpdaterStatus(listener)   // Subscribe to updater events
window.apoAPI.backupUserData()            // Export user data zip
window.apoAPI.restoreUserData()           // Restore from zip

window.windowAPI.minimize()
window.windowAPI.maximize()
window.windowAPI.close()
```

**Never call `require('electron')` or `require('fs')` from `src/` files** — they run in the renderer process and must use the bridge.

---

## Central Patterns — READ BEFORE TOUCHING ANYTHING

### markDirty() — the hub for all EQ changes

Every time the EQ state changes (filter added/edited/removed, preamp moved, etc.) you **must** call `markDirty()`. It does three things:
1. Sets `appState.dirty = true`
2. Schedules auto-save to APO (if auto-save mode is on)
3. Schedules an auto-snapshot (debounced 8 s)

Do **not** call `updateStatus`, `scheduleAutoSnapshot`, or `scheduleAutoApply` directly — always go through `markDirty()`.

### saveConfig() — saving to APO

`async function saveConfig(options = {})` at line ~3453 in `src/main.js`.  
- Serializes the current config via `src/config/serializer.js`
- Writes the result to the APO `config.txt` via `window.apoAPI.writeConfig`
- Called by the Save button, `Ctrl+S`, and auto-save timer

### Undo/Redo stack


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IJustItay/Neon-Equalizer](https://github.com/IJustItay/Neon-Equalizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->

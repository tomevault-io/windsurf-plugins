---
trigger: always_on
description: Build plugins for Ulanzi D200H / D200 / D200X stream controllers using the Ulanzi Studio SDK.
---

# GitHub Copilot Instructions - Ulanzi Studio Plugin Development
# Place this file at .github/copilot-instructions.md in your project.
# Reference: https://github.com/ukinora/ulanzi-studio-plugin-skill

# Ulanzi Studio Plugin Development

Build plugins for Ulanzi D200H / D200 / D200X stream controllers using the Ulanzi Studio SDK.

TRIGGER when: user asks to create a Ulanzi Studio plugin/app, build a stream deck action, or modify an existing Ulanzi plugin.

## Plugin Architecture

Two runtime modes (choose one per plugin):

| Mode | Entry Point | Use Case |
|------|------------|----------|
| **HTML** | `plugin/app.html` | Canvas drawing, UI-driven, simpler setup |
| **Node.js** | `plugin/app.js` | System access, file I/O, complex logic |

## Directory Structure

```
com.ulanzi.{name}.ulanziPlugin/
  manifest.json                      # Plugin metadata (REQUIRED)
  plugin/
    app.html                         # Entry point (HTML mode)
    app.js                           # Main logic
    actions/
      {action}.js                    # Action class
  property-inspector/
    config/
      inspector.html                 # Settings UI
      inspector.js                   # Settings logic
  libs/
    js/
      constants.js                   # Event constants
      eventEmitter.js                # Pub/sub system
      timers.js                      # Web Worker timers
      utils.js                       # Utilities (fetch, form, canvas)
      ulanzideckApi.js               # SDK API ($UD object)
    css/
      udpi.css                       # Property Inspector styling
    assets/                          # UI icons (svg)
  assets/
    icons/
      icon.png                       # Plugin icon (72x72)
      categoryIcon.png               # Category icon (72x72)
      actionDefaultImage.png         # Default key image (196x196)
  en.json                            # English localization
  ko_KR.json                         # Korean localization
  zh_CN.json                         # Chinese localization
```

## manifest.json Schema

```json
{
  "Version": "1.0.0",
  "Author": "AuthorName",
  "Name": "Plugin Name",
  "Description": "Short description",
  "Icon": "assets/icons/icon.png",
  "Category": "Category Name",
  "CategoryIcon": "assets/icons/categoryIcon.png",
  "CodePath": "plugin/app.html",
  "Type": "JavaScript",
  "SupportedInMultiActions": false,
  "PrivateAPI": true,
  "UUID": "com.ulanzi.ulanzideck.{pluginName}",
  "Actions": [
    {
      "Name": "Action Name",
      "Icon": "assets/icons/icon.png",
      "PropertyInspectorPath": "property-inspector/config/inspector.html",
      "state": 0,
      "States": [
        { "Image": "assets/icons/actionDefaultImage.png" }
      ],
      "Tooltip": "Action tooltip",
      "UUID": "com.ulanzi.ulanzideck.{pluginName}.{actionName}",
      "SupportedInMultiActions": false
    }
  ],
  "OS": [
    { "Platform": "windows", "MinimumVersion": "10" },
    { "Platform": "mac", "MinimumVersion": "10.11" }
  ],
  "Software": { "MinimumVersion": "6.1" }
}
```

**UUID rules:**
- Plugin UUID: 4 dot-separated segments (`com.ulanzi.ulanzideck.myPlugin`)
- Action UUID: 5+ segments (`com.ulanzi.ulanzideck.myPlugin.myAction`)

## SDK API Reference ($UD)

### Connection

```javascript
$UD.connect('com.ulanzi.ulanzideck.myPlugin')  // Plugin main service
$UD.connect('com.ulanzi.ulanzideck.myPlugin.myAction')  // Property inspector
```

### Event Handlers

| Method | Fires When | Callback Data |
|--------|-----------|---------------|
| `onConnected(fn)` | WebSocket connected | `{}` |
| `onAdd(fn)` | Action added to key | `{ context, param }` |
| `onRun(fn)` | Key pressed | `{ context }` |
| `onSetActive(fn)` | Action becomes visible | `{ context, active }` |
| `onClear(fn)` | Action removed | `{ param: [{ context }] }` |
| `onParamFromApp(fn)` | Settings received from app | `{ context, param }` |
| `onParamFromPlugin(fn)` | Settings received from PI | `{ context, param }` |
| `onClose(fn)` | Connection closed | `{}` |
| `onError(fn)` | WebSocket error | error string |
| `onSelectdialog(fn)` | File/folder dialog result | selection result |

### Icon Methods

```javascript
// Base64 canvas image (most common for dynamic content)
$UD.setBaseDataIcon(context, base64DataUrl, optionalText)

// Predefined state from manifest States array
$UD.setStateIcon(context, stateIndex, optionalText)

// Local image file path
$UD.setPathIcon(context, './assets/icons/myIcon.png', optionalText)

// Animated GIF (base64 or path)
$UD.setGifDataIcon(context, base64GifData, optionalText)
$UD.setGifPathIcon(context, './assets/myAnim.gif', optionalText)
```

### Settings & Communication

```javascript
// Send settings to main app (from Property Inspector)
$UD.sendParamFromPlugin(settingsObject, optionalContext)

// UI helpers
$UD.toast('Notification message')
$UD.openUrl('https://example.com')
$UD.openView('./popup.html', 300, 200)
$UD.selectFileDialog('image(*.jpg *.png)')
$UD.selectFolderDialog()

// Localization
$UD.t('keyName')  // Returns translated string
```

## Plugin Main Service Pattern (app.js)

```javascript
const ACTION_CACHES = {}
$UD.connect('com.ulanzi.ulanzideck.{pluginName}')
$UD.onConnected(conn => {})

$UD.onAdd(jsn => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ukinora) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

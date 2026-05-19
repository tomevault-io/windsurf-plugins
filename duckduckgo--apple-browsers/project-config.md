---
trigger: always_on
description: Rules for creating and maintaining pixel definition JSON5 files that document pixels fired by the iOS and macOS apps
---

# Pixel Registry Definitions

## Overview

Pixel definitions are JSON5 files that document pixels and wide events fired by the iOS and macOS apps. They live in:

- **iOS:** `iOS/PixelDefinitions/pixels/definitions/*.json5`
- **macOS:** `macOS/PixelDefinitions/pixels/definitions/*.json5`
- **iOS wide events:** `iOS/PixelDefinitions/wide_events/definitions/*.json5`

Each platform has its own `params_dictionary.json5` and `suffixes_dictionary.json5` for reusable definitions.

**Note:** Each definitions directory contains a `TEMPLATE.json5` file. These are scaffolds for creating new definition files — they are not real pixel definitions. Ignore them when reviewing or auditing existing definitions (their placeholder `expires` dates are intentional examples).

## Pixel Definition Structure

Each `.json5` file is a JSON5 object where keys are pixel names and values describe the pixel:

```json5
{
    "pixel_name_here": {
        "description": "When and why this pixel fires",
        "owners": ["githubUsername"],
        "triggers": ["other"],
        "suffixes": ["first_daily_count", "platform", "form_factor"],
        "parameters": ["appVersion", "errorCode", "errorDomain"],
        // Only for temporary pixels — omit for permanent ones
        "expires": "2025-06-30"
    }
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | When the pixel fires and its purpose |
| `owners` | string[] | GitHub usernames of responsible people |
| `triggers` | string[] | What causes the pixel to fire (see trigger values below) |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `suffixes` | array | Dynamic parts appended to the pixel name |
| `parameters` | array | Query parameters sent with the pixel |
| `expires` | string | ISO date (`YYYY-MM-DD`) for temporary pixels |

### Trigger Values

Valid trigger values: `"other"`, `"scheduled"`, `"startup"`, `"page_load"`, `"new_tab"`, `"exception"`, `"user_submitted"`, `"search_ddg"`.

Most pixels use `"other"`. Use `"scheduled"` for daily/periodic pixels, `"startup"` for app-launch pixels, and `"page_load"` for navigation-related pixels.

## Determining Parameters from Swift Code

Pixel definitions must document **all** query parameters sent over the wire, including default ones. To determine the correct parameters:

### Always-included Parameters

**`appVersion`** is added by default to every pixel by PixelKit. Include `"appVersion"` in every definition, unless the pixel call disables it.

**`pixelSource`** is automatically added only if the pixel's `standardParameters` property returns `[.pixelSource]`. Check the pixel event's `standardParameters` computed property in Swift — if it returns `[.pixelSource]`, include `"pixelSource"` in the definition.

### Error Parameters

If the pixel event carries an `Error` (via associated value or the `error` property), PixelKit automatically extracts and sends:
- `errorCode` (key: `"e"`) and `errorDomain` (key: `"d"`)
- `underlyingErrorCode` (key: `"ue"`) and `underlyingErrorDomain` (key: `"ud"`) if present

Include these dictionary references in the definition when the pixel carries error information.

### Custom Parameters

Check the pixel event's `parameters` computed property in Swift for any additional parameters. Also inspect the call site where the pixel is fired — look for `withAdditionalParameters:` arguments and trace any helper functions that build those parameters. These are pixel-specific and must be included in the definition (either as dictionary references or inline objects).

### Where to Look in Swift

- **iOS:** `iOS/Core/PixelEvent.swift` defines pixel names. `iOS/Core/Pixel.swift` has `PixelParameters` constants. Check the `parameters` and `standardParameters` properties on the pixel event enum.
- **macOS:** `macOS/DuckDuckGo/Statistics/GeneralPixel.swift` defines many pixel names, parameters, and standard parameters. However, pixel events can also be defined in dedicated files (e.g. `UpdateFlowPixels.swift`, `CrashReportPixels.swift`) — search for types conforming to `PixelKitEvent`.
- **Shared:** `SharedPackages/BrowserServicesKit/Sources/PixelKit/` contains `PixelKit.swift` (firing logic) and `PixelKitEvent.swift` (protocol).

## Reusing Parameters from the Dictionary

`params_dictionary.json5` defines common parameters. Reference them by key name as a string:

```json5
"parameters": [
    "appVersion",       // Reuses definition from params_dictionary.json5
    "errorCode",        // key: "e", type: integer
    "errorDomain",      // key: "d", type: string
    "underlyingErrorCode",
    "underlyingErrorDomain"
]
```

To define a custom inline parameter, use an object instead:

```json5
"parameters": [
    "appVersion",
    {
        "key": "customParam",
        "type": "string",
        "description": "What this parameter represents",
        "enum": ["value1", "value2"]
    }
]
```

### Parameter Object Fields

- `key` — the actual query parameter key sent in the pixel (use this for fixed keys)
- `keyPattern` — regex pattern for dynamic keys (e.g. `"^ue[0-9]?$"` for `ue`, `ue0`, `ue1`, etc.)
- `type` — `"string"`, `"integer"`, `"number"`, or `"boolean"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

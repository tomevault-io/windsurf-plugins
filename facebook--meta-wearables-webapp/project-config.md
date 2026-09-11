---
trigger: always_on
description: > Developer docs: [https://wearables.developer.meta.com/docs/develop/webapps](https://wearables.developer.meta.com/docs/develop/webapps)
---

# Meta Wearables Web Apps — AI Instructions

> Developer docs: [https://wearables.developer.meta.com/docs/develop/webapps](https://wearables.developer.meta.com/docs/develop/webapps)
>
> Web Apps docs MCP: [https://mcp.developer.meta.com/wearables](https://mcp.developer.meta.com/wearables)
>
> MCP tool: `search_webapps_docs`
>
> Auth: no auth, OAuth, tokens, or custom authorization headers are required.

If your AI tool supports MCP, configure `https://mcp.developer.meta.com/wearables` as a remote HTTP MCP server and call `search_webapps_docs` for current Web Apps documentation. If MCP is unavailable, use the developer docs URL above directly.

## Design & Performance Constraints

All webapps target the Meta Display Glasses — a 600x600dp additive waveguide display with no touchscreen.

### Display
- **Viewport:** `<meta name="viewport" content="width=600, height=600, initial-scale=1.0">`
- **Description:** `<meta name="description" content="...">` in the `<head>` with a brief, app-specific summary of what the app does.
- **MRBD identification:** `<meta name="mrbd-web-app-capable" content="yes">` in the `<head>` to positively identify the page as a Meta Display Glasses (MRBD) compatible webapp (keep `content="yes"` verbatim).
- **Additive display:** Black (#000000) is transparent. Use dark gray (#1C1E21) as background, white (#FFFFFF) for text/icons.
- **Safe zone:** 8dp margin all sides (584x584dp usable). Header: 24dp from top, 64dp tall. Button height: 88dp.
- **Typography:** H1 28dp bold, H2 22dp bold, Body 16dp, Body2 14dp, Meta 12dp. Min 14dp for interactive elements.
- **Colors:** #FFFFFF primary, #E4E6EB secondary, #B0B3B8 muted, #1C1E21 background. 4.5:1 contrast ratio minimum.

### Input
- **D-pad (captouch):** Arrow keys navigate focus between elements. Enter (or an EMG pinch) activates the focused element.
- **Pinch = activate:** An EMG pinch fires Enter/click on the focused element — not a positioned click. Continuous drag is opt-in via `body { touch-action: none; }` in the initial CSS — see the add-gestures skill.
- **Back:** A thumb + middle-finger back gesture (or Escape) returns to the previous screen — no back button needed.
- **Text entry:** Standard HTML text fields open the on-glasses handwriting + voice composer on focus + tap — see the add-text-input skill.
- **No cursor, no touch:** Focus-based navigation only. All elements must be reachable via sequential D-pad navigation.
- **Interaction states:** Idle (scale 1x, 80% opacity) → Focused (scale -8dp, 100% opacity) → Pressed.
- Keep navigation shallow — 3 steps or fewer to any action.

### Performance
- **Targets:** <3s load, <500KB JS gzipped, 60fps, <128MB memory, <10 network requests.
- **Code:** Vanilla JS or lightweight frameworks. No continuous intervals when idle. CSS transitions over JS animations.
- **Assets:** Unicode/inline PNGs for icons. No external fonts. Inline assets <2KB as data URIs.
- **Offline:** Cache with Service Worker. Show UI immediately. Handle fetch failures gracefully.


# Add Device Sensors to Meta Display Glasses WebApp

Add IMU and GPS sensor integration to an existing webapp using standard Web APIs. No SDK required.

The glasses expose sensor data through two API families:
- **DeviceMotionEvent / DeviceOrientationEvent** — IMU data (accelerometer, gyroscope, compass heading, tilt)
- **navigator.geolocation** — GPS location from the paired companion phone

## Permissions: wait for a user action

Request permission and start sensor/location updates **only** from an explicit user action (a Start/Enable button) — never on load, `init()`, or screen entry (`requestPermission()` only resolves inside a user gesture). If permission isn't granted, show a message and stop: don't add listeners, start a watch, or auto-retry.

## Prerequisites

- Existing webapp created via `/create-webapp`

## Available Sensors

### Motion & Orientation (IMU)

The glasses IMU provides high-frequency updates with low latency via two event-based APIs:

**DeviceOrientationEvent** — fires continuously as the glasses rotate:

| Property | Type | Range | Description |
|----------|------|-------|-------------|
| `alpha` | number | 0–360° | Rotation around Z axis (compass heading). 0 = North. |
| `beta` | number | −180° to 180° | Rotation around X axis (front-to-back tilt). |
| `gamma` | number | −90° to 90° | Rotation around Y axis (left-to-right tilt). |
| `absolute` | boolean | — | true if orientation is relative to the Earth's coordinate frame. |

**DeviceMotionEvent** — fires at a regular interval with accelerometer and gyroscope data:

| Property | Unit | Description |
|----------|------|-------------|
| `accelerationIncludingGravity.x/y/z` | m/s² | Linear acceleration including gravitational force. |
| `acceleration.x/y/z` | m/s² | Linear acceleration with gravity removed (may be null). |
| `rotationRate.alpha/beta/gamma` | deg/s | Gyroscope rotation rate around each axis. |
| `interval` | ms | Time interval between events. |

### Geolocation (GPS)

Location is fetched from the paired companion phone — the glasses have no GPS hardware. Permission is granted automatically by the glasses host app.

| Property | Type | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebook/meta-wearables-webapp](https://github.com/facebook/meta-wearables-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->

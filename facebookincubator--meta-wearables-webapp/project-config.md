---
trigger: always_on
description: > Full API reference: [https://wearables.developer.meta.com/llms.txt?full=true](https://wearables.developer.meta.com/llms.txt?full=true)
---

# Meta Wearables Web Apps — AI Instructions

> Full API reference: [https://wearables.developer.meta.com/llms.txt?full=true](https://wearables.developer.meta.com/llms.txt?full=true)
>
> Developer docs: [https://wearables.developer.meta.com/docs/develop/](https://wearables.developer.meta.com/docs/develop/)

## Design & Performance Constraints

All webapps target the Meta Display Glasses — a 600x600dp additive waveguide display with no touchscreen.

### Display
- **Viewport:** `<meta name="viewport" content="width=600, height=600, initial-scale=1.0">`
- **Additive display:** Black (#000000) is transparent. Use dark gray (#1C1E21) as background, white (#FFFFFF) for text/icons.
- **Safe zone:** 8dp margin all sides (584x584dp usable). Header: 24dp from top, 64dp tall. Button height: 88dp.
- **Typography:** H1 28dp bold, H2 22dp bold, Body 16dp, Body2 14dp, Meta 12dp. Min 14dp for interactive elements.
- **Colors:** #FFFFFF primary, #E4E6EB secondary, #B0B3B8 muted, #1C1E21 background. 4.5:1 contrast ratio minimum.

### Input
- **D-pad (captouch):** Arrow keys navigate focus between elements. Enter selects. Escape goes back.
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
|----------|------|-------------|
| `latitude` | number | Latitude in decimal degrees. |
| `longitude` | number | Longitude in decimal degrees. |
| `accuracy` | number | Accuracy of lat/lon in metres. |
| `altitude` | number \| null | Altitude in metres above sea level. |
| `altitudeAccuracy` | number \| null | Accuracy of altitude in metres. |
| `speed` | number \| null | Speed in m/s. |
| `heading` | number \| null | Direction of travel in degrees from North. |

Accuracy depends on the companion phone's GPS and network fix. Expect 5–50 m typical accuracy. The first call may take several seconds while the phone acquires a fix.

## Steps

### 1. Gather Information

Ask the user:
- **Which sensors?** Motion (accelerometer/gyroscope), orientation (compass/tilt), geolocation (GPS)?
- **What for?** Compass, level tool, step counter, head tracking, spatial AR, location display?
- **Continuous or one-shot?** (for geolocation: `watchPosition` vs `getCurrentPosition`)

### 2. Add Sensor Display UI

Add to the appropriate screen in `index.html`:

```html
<div class="sensor-panel">
  <div class="data-grid">
    <div class="card">
      <div class="card-subtitle">X-Axis</div>
      <div class="card-value" id="sensor-x">0.00</div>
    </div>
    <div class="card">
      <div class="card-subtitle">Y-Axis</div>
      <div class="card-value" id="sensor-y">0.00</div>
    </div>
    <div class="card">
      <div class="card-subtitle">Z-Axis</div>
      <div class="card-value" id="sensor-z">0.00</div>
    </div>
    <div class="card">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebookincubator/meta-wearables-webapp](https://github.com/facebookincubator/meta-wearables-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->

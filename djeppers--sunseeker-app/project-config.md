---
trigger: always_on
description: Build a PebbleOS watchapp called **SunSeeker** that helps users locate where the sun is rising and setting. The app uses the compass sensor to show real-time directional guidance - point your wrist toward sunrise or sunset. It combines GPS location data (from the paired phone), the watch's magnetometer/compass, time of day, and a solar position algorithm to give you a live compass view with sunrise/sunset bearings.
---

# SunSeeker - PebbleOS Sun Tracker App

## Project Overview

Build a PebbleOS watchapp called **SunSeeker** that helps users locate where the sun is rising and setting. The app uses the compass sensor to show real-time directional guidance - point your wrist toward sunrise or sunset. It combines GPS location data (from the paired phone), the watch's magnetometer/compass, time of day, and a solar position algorithm to give you a live compass view with sunrise/sunset bearings.

## Target SDK

Use the **Alloy** JavaScript SDK (the new Pebble JS framework powered by Moddable, released Feb 2026). Do NOT use the old C SDK or Rocky.js. Alloy lets you write the entire app in modern JavaScript (ES2025) that runs natively on the watch via the XS engine.

Key Alloy references:
- Official examples: https://github.com/Moddable-OpenSource/pebble-examples
- Alloy docs: https://developer.repebble.com/guides/alloy/
- Sensor APIs follow the ECMA-419 Sensor Class Pattern
- UI uses the **Piu** framework (declarative, component-based) or **Poco** (low-level graphics)
- CloudPebble (browser IDE): https://cloudpebble.repebble.com/

## Target Platforms

- **emery** (Pebble Time 2) - 200×228 px, color, compass sensor
- **gabbro** (Pebble Round 2) - 260×260 px, color, round display

The app should work on both. Use `screen` from `pebble` to detect dimensions.

## Architecture

### Module Structure

```
sunseeker/
├── package.json                    # Pebble project config
├── src/
│   ├── embeddedjs/
│   │   ├── manifest.json           # Alloy build manifest
│   │   ├── main.js                 # App entry point, UI, sensor wiring
│   │   └── sun.js                  # Solar position calculator (pure math, no Pebble deps)
│   └── pkjs/
│       └── index.js                # PebbleKit JS (phone-side, for location proxy)
├── tests/
│   └── sun.test.js                 # Unit tests for solar calculator (runs in Node.js)
└── CLAUDE.md                       # This file
```

### 1. Solar Position Calculator (`sun.js`)

This module must be **pure math with zero Pebble dependencies** so it can be unit-tested in Node.js.

Export these functions:

#### `sunTimes(lat, lon, date) → { sunrise: Date, sunset: Date, polarDay: bool, polarNight: bool }`
Calculate sunrise and sunset times for a given location and date.
- Use the standard solar position equations (Julian day, solar declination, hour angle)
- Standard zenith of 90.833° (accounts for atmospheric refraction + solar disc radius)
- Handle polar edge cases (midnight sun / polar night) gracefully
- Return local times adjusted for timezone

#### `sunAzimuth(lat, lon, date) → { sunrise: number, sunset: number, current: number, elevation: number }`
Calculate the compass bearing (0-360°, 0=North, 90=East) for:
- Sunrise azimuth (the bearing where the sun rose/will rise)
- Sunset azimuth (the bearing where the sun sets/will set)
- Current sun azimuth (where the sun is right now)
- Current sun elevation (degrees above/below horizon, negative = below)

#### `solarNoon(lat, lon, date) → Date`
Return the time of solar noon for the location.

#### `dayLength(lat, lon, date) → number`
Return day length in minutes.

Algorithm notes:
- Julian Day from calendar date
- Solar mean anomaly and ecliptic longitude
- Solar declination from obliquity of the ecliptic
- Hour angle for sunrise/sunset from latitude + declination
- Azimuth from hour angle + declination + latitude
- Equation of time for solar noon correction

### 2. Main App (`main.js`)

#### Sensors (ECMA-419 pattern)
All Pebble sensors use the ECMA-419 Sensor Class Pattern. Reference the examples at:
- `hellolocation/main.js` for GPS location
- `piu/apps/compass/main.js` for compass heading

```js
// Compass - heading in degrees from magnetic north
import Compass from "pebble/sensor/Compass";
const compass = new Compass();
compass.onreading = function() {
    const heading = compass.heading; // degrees, 0-360
};
compass.start();

// Location - GPS from paired phone
import Location from "pebble/sensor/Location";
const location = new Location();
location.onreading = function() {
    const lat = location.latitude;
    const lon = location.longitude;
};
location.start();
```

**Important**: These import paths are based on the Alloy examples. Before writing the code, check the actual examples at `piu/apps/compass` and `hellolocation` for the correct import syntax. The sensor pattern uses `.onreading` callbacks and `.start()` / `.stop()` methods per ECMA-419.

#### UI Design (Piu framework)

The app should show a **compass rose view** with:

1. **Compass ring** - rotating ring showing N/S/E/W that moves with the watch's compass heading, so North always points to actual north
2. **Sun position indicators**:
   - A sunrise marker (☀ or a wedge/arc in warm yellow/orange) drawn at the sunrise azimuth bearing
   - A sunset marker (similar, in deeper orange/red) drawn at the sunset azimuth bearing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djeppers/sunseeker_app](https://github.com/djeppers/sunseeker_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->

---
trigger: always_on
description: Quick reference for implementing navigation and routing with Mapbox Directions API and Navigation SDKs.
---

# Mapbox Navigation Patterns

Quick reference for implementing navigation and routing with Mapbox Directions API and Navigation SDKs.

## Product Decision

| Need                          | Solution                                               |
| ----------------------------- | ------------------------------------------------------ |
| **Show a route on a web map** | Directions API                                         |
| **Turn-by-turn iOS**          | Navigation SDK for iOS (SwiftUI + drop-in NVC default) |
| **Turn-by-turn Android**      | Navigation SDK for Android                             |
| **Voice guidance**            | Navigation SDK only                                    |
| **Multi-stop optimization**   | Optimization API                                       |

## Directions API (Web)

Coordinates are always `longitude,latitude` order. Default to the `driving-traffic` profile — it factors in live traffic, congestion, and incidents. Use `driving` only when you need `arrive_by` (not supported by `driving-traffic`); both profiles support `depart_at`.

### Basic Route

```javascript
const query = await fetch(
  `https://api.mapbox.com/directions/v5/mapbox/driving-traffic/` +
    `${start[0]},${start[1]};${end[0]},${end[1]}?` + // lon,lat
    `steps=true&geometries=geojson&access_token=${token}`
);

const route = (await query.json()).routes[0];

// Display on map
map.addSource('route', {
  type: 'geojson',
  data: { type: 'Feature', geometry: route.geometry }
});

map.addLayer({
  id: 'route',
  type: 'line',
  source: 'route',
  paint: {
    'line-color': '#3b9ddd',
    'line-width': 8
  }
});
```

### Alternative Routes

```javascript
// Add alternatives=true
const url = `...&alternatives=true&...`;

const routes = json.routes; // Returns multiple routes

// Main route = routes[0], alternatives = routes[1], routes[2]
```

### Multi-Stop Routing

```javascript
// Up to 25 waypoints
const waypoints = [start, stop1, stop2, stop3, end];
const coords = waypoints.map((w) => `${w[0]},${w[1]}`).join(';'); // lon,lat

const url = `https://api.mapbox.com/directions/v5/mapbox/driving-traffic/${coords}?...`;
```

### Route Optimization

```javascript
// Optimize waypoint order — hard limit: 12 coordinates max (v1 API)
// source/destination only accept 'first'/'any' and 'last'/'any' — no numeric indices
const url =
  `https://api.mapbox.com/optimized-trips/v1/mapbox/driving-traffic/${coords}?` +
  `source=first&destination=last&roundtrip=true&...`;

const optimized = json.trips[0];
const order = json.waypoints.map((wp) => wp.waypoint_index);

// More than 12 stops, or need time windows/vehicle capacities? See Optimization
// API v2 (separate async job-submission API, Public Beta, up to 1,000 locations)
```

### Congestion-Based Route Coloring

```javascript
// annotations must be paired with overview=full or the geometry won't line up
// point-for-point with the per-segment annotation array
const url =
  `https://api.mapbox.com/directions/v5/mapbox/driving-traffic/${coords}?` +
  `overview=full&annotations=duration,distance,congestion&...`;

// Color by congestion level
const congestion = route.legs[0].annotation.congestion;
// Values: 'low', 'moderate', 'heavy', 'severe', 'unknown'
```

### Turn-by-Turn Instructions

```javascript
const steps = route.legs[0].steps;

steps.forEach((step) => {
  console.log(step.maneuver.instruction); // "Turn left onto Main St"
  console.log(step.distance); // meters
  console.log(step.duration); // seconds
});
```

## Navigation SDK for iOS

**Default:** SwiftUI app shell + wrap `NavigationViewController` with `UIViewControllerRepresentable` (official getting-started). Fully custom Core UI ([CoreSDKExample](https://github.com/mapbox/mapbox-navigation-ios/tree/main/Examples/CoreSDKExample)) only when explicitly requested.

**Setup first:** SPM (`MapboxNavigationCore` + `MapboxNavigationUIKit`), `.netrc` download token, `MBXAccessToken`, location permissions, background `audio`/`location` — see `references/ios-navigation-sdk.md` checklist and [install guide](https://docs.mapbox.com/ios/navigation/guides/install/).

For specialized topics (road cameras, history, e-horizon, CarPlay, offline, styled chrome, etc.), use the **Example patterns catalog** in `references/ios-navigation-sdk.md`. Load `references/ios-navigation-specialized.md` for multi-stop, route line, camera, road cameras, and route alerts. Do not fetch upstream sample source unless the user asks to open a specific example.

**Sample host ≠ API stack:** `AdditionalExamples` are often UIKit demos. APIs on `NavigationMapView` (waypoints, final-waypoint image, route line, camera, callouts, road cameras) are stack-independent — wrap `NavigationMapView` in `UIViewRepresentable`. Road cameras: `navigationMapView.mapView.mapboxMap` + `RoadCamerasManager(navigatorHandle: provider.navigatorHandle)`. True UIKit-only: NVC chrome (top/bottom bars, styled UI elements, embed NVC).

### Default: SwiftUI + drop-in NavigationViewController

```swift
import MapboxNavigationCore
import MapboxNavigationUIKit
import SwiftUI

struct NavigationViewControllerWrapper: UIViewControllerRepresentable {
    let navigationRoutes: NavigationRoutes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mapbox/mapbox-agent-skills](https://github.com/mapbox/mapbox-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

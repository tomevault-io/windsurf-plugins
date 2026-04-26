---
trigger: always_on
description: Baseline.js is a **reference implementation** and **showcase project** for building custom user interfaces (skins) for [Streamline-Bridge](https://github.com/tadelv/reaprime). This project demonstrates best practices for integrating with the Bridge API and creating an intuitive, visually appealing espresso machine interface.
---

# Claude Development Guide for Baseline.js

## Project Overview

Baseline.js is a **reference implementation** and **showcase project** for building custom user interfaces (skins) for [Streamline-Bridge](https://github.com/tadelv/reaprime). This project demonstrates best practices for integrating with the Bridge API and creating an intuitive, visually appealing espresso machine interface.

**Primary Purpose**: To serve as a learning resource and starting point for developers who want to build their own custom skins for Streamline-Bridge.

## Core Architecture

### Technology Stack

- **Framework**: Next.js 16 with App Router
- **UI Library**: React 19
- **Language**: TypeScript
- **Styling**: Tailwind CSS 4
- **Components**: Radix UI for accessible primitives
- **Visualization**: HTML5 Canvas API

### Application Structure

The application is built as a single-page application (SPA) with the main logic embedded in `app/page.tsx`. This unconventional approach uses `dangerouslySetInnerHTML` to render a self-contained HTML/CSS/JavaScript application within a React component.

**Why this approach?**
- Single-file deployment for easy distribution
- Minimal dependencies at runtime
- Can be easily extracted and used standalone
- Demonstrates that Streamline-Bridge skins don't require complex tooling

### State Management

The application uses vanilla JavaScript state management (`STATE` object) rather than React hooks, since the core application logic is embedded HTML. State includes:

- **screen** - Current UI screen (main, sleep, carousel, brewing, done, settings)
- **machineState** - Current machine status from Bridge API
- **machineReady** - Whether machine is ready to brew
- **carouselStep** - Current step in brewing workflow
- **profiles** - Available brewing profiles from machine
- **WebSocket connections** - For real-time data (machine snapshots, scale readings)

## Streamline-Bridge Integration

### API Communication

All communication with Streamline-Bridge happens through:

1. **REST API** (`CONFIG.apiUrl` - default: `http://localhost:8080`)
   - `/api/v1/machine/state` - Get machine status
   - `/api/v1/profiles` - List available profiles
   - `/api/v1/workflow` - Set active profile
   - `/api/v1/workflow/start` - Start brewing
   - `/api/v1/devices` - List connected devices (scale, etc.)
   - `/api/v1/devices/scan?connect=true` - Scan for and connect devices

2. **WebSocket Endpoints**
   - `/ws/v1/machine/snapshot` - Real-time machine data (pressure, flow)
   - `/ws/v1/scale/snapshot` - Real-time scale weight readings

### Key Integration Points

#### Machine State Monitoring
```javascript
// Polls every 1 second
setInterval(async () => {
  await getMachineState();
  // Auto-transitions based on state changes
}, 1000);
```

#### Profile Management
When setting a profile, the full profile object must be sent, not just the ID:
```javascript
const profile = STATE.profiles.find(p => p.id === profileId);
await apiCall('/api/v1/workflow', {
  method: 'PUT',
  body: JSON.stringify({ profile: profile.profile })
});
```

#### Scale Integration
The app automatically scans for and connects to scales when entering the brewing workflow. Real-time weight is displayed during the "Weigh Coffee" step.

## Development Guidelines

### When Making Changes

1. **Preserve the Reference Nature**: This is meant to be a simple, educational example. Avoid over-engineering.

2. **API Compatibility**: Always check the [Streamline-Bridge API documentation](https://github.com/tadelv/reaprime) when modifying API calls.

3. **Visual Consistency**: The design uses a dark theme with blue accents. Maintain this aesthetic or document intentional departures.

4. **Performance**: Canvas animations should be smooth. Use requestAnimationFrame and be mindful of particle counts.

5. **Browser Compatibility**: The app should work in modern browsers and the Streamline-Bridge embedded view.

### Common Tasks

#### Adding a New Screen
1. Create a render function (e.g., `renderNewScreen()`)
2. Add the screen name to the STATE object
3. Update the main `render()` function to handle the new screen
4. Add navigation logic to transition to/from the screen

#### Modifying the Workflow
The carousel steps are defined in the `steps` array within `renderCarousel()`. Each step has:
- `title` - Step name
- `description` - Instructions for the user
- `hasInput` - Whether the step requires user input
- `showWeight` - Whether to display scale readings

#### Adding New Settings
1. Update the `CONFIG` object with default value
2. Add localStorage save/load in `saveSettings()`
3. Add input field in `renderSettingsScreen()`
4. Update `saveSettingsFromDialog()` to read and save the new setting

### Canvas Animations

Three distinct animations are used:

1. **Ambient Animation** (`renderAmbientAnimation`) - Subtle particle effects on main screens
2. **Brewing Visualization** (`renderBrewingVisualization`) - Data-driven particles responding to pressure/flow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tadelv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->

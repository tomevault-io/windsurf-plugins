---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Zoom Video SDK UI Toolkit is a prebuilt video chat user interface powered by the Zoom Video SDK. It's a React-based library that provides ready-to-use components for video conferencing functionality.

**Package Distribution**: Built as both ESM and UMD modules (`videosdk-ui-toolkit.min.esm.js` and `videosdk-ui-toolkit.min.umd.js`) with accompanying CSS (`videosdk-ui-toolkit.css`).

## Development Commands

### Setup
```bash
# First time setup - create dev config from template
cp src/config/devTemplate.ts src/config/dev.ts

# Install dependencies and start dev server
npm i && npm run dev
```

### [Https](https://github.com/FiloSottile/mkcert)

## Windows

`choco install mkcert`

## Mac

`brew install mkcert`

`brew install nss` # if you use Firefox

### Generate certificate

`mkcert localhost 127.0.0.1 ::1`

### Build & Package
```bash
# Standard production build
npm run build

# Build and package as .tgz
npm run tgz

# Development with HTTPS (uses localhost+2.pem and localhost+2-key.pem)
npm run https

# Bundle analysis
npm run visualizer
```

### Testing(WIP)
```bash
# Run tests in watch mode
npm run test:unit

# Run tests once with coverage
npm run test:coverage

# CI test run
npm run test:ci
```

### Code Quality
```bash
# Lint TypeScript/React files
npm run lint

# Lint and auto-fix
npm run lint:fix

# Lint SCSS/CSS (allows up to 4000 warnings)
npm run lint:style
```

### Documentation
```bash
# Generate TypeDoc documentation
npm run docs
```

## Architecture

### State Management (Redux Toolkit)
The application uses Redux Toolkit for centralized state management. The store is configured in `src/store/store.ts` with feature-based slices:

- **session**: Session state and connection status (src/store/sessionSlice.ts)
- **ui**: UI state, view modes, popper states, layout customization (src/store/uiSlice.ts)
- **participant**: Participant management and user lists (src/features/participant/participantSlice.ts)
- **chat**: Chat messages and state (src/features/chat/chatSlice.ts)
- **setting**: Settings, devices, quality statistics (src/features/setting/settingSlice.ts)
- **caption**: Caption/translation state (src/features/caption/captionSlice.ts)
- **media**: Media stream state (src/features/media/mediaSlice.ts)
- **subsession**: Breakout room functionality (src/features/subsession/subsessionSlice.ts)
- **whiteboard**: Whiteboard state (src/features/whiteboard/whiteboardSlice.ts)

### Public API Entry Point
`src/uikit/index.tsx` exports the singleton UIToolkitAPI that consumers use. Key methods:
- `joinSession()` / `closeSession()` / `leaveSession()` - Session lifecycle
- `openPreview()` / `closePreview()` - Preview flow before joining
- `showChatComponent()` / `showUsersComponent()` / `showSettingsComponent()` - Component rendering
- `on()` / `off()` - Event listener management (proxies to Zoom Video SDK events)
- `migrateConfig()` - Converts legacy config to current format
- `getAllUser()` / `getCurrentUserInfo()` / `getSessionInfo()` - Data accessors

### UIToolkit Core Class
`src/uikit/UIToolkit.tsx` manages the React rendering lifecycle:
- Creates a singleton Zoom Video SDK client (`ZoomVideo.createClient()`)
- Manages React roots for the main app and individual components (chat, users, settings, controls)
- Enforces singleton pattern for components - only one instance of each component type can be active
- Handles cleanup on session close/destruction

### Feature Structure
Each feature in `src/features/` follows a consistent pattern:
- **Components**: React components for UI (`components/` subdirectory)
- **Slice**: Redux state management (`*Slice.ts`)
- **Hooks**: Custom React hooks for feature logic (`hooks/` subdirectory)
- **Constants**: Feature-specific constants (`*-constants.ts` or `*-constant.ts`)
- **Utils**: Helper functions (`*-utils.ts`)
- **Types**: TypeScript type definitions (`*-types.d.ts`)

Key features:
- **audio**: Microphone, speaker, phone dial-in
- **video**: Camera, video rendering, gallery/speaker view
- **share**: Screen sharing and annotation
- **chat**: In-session messaging
- **participant**: User management, host controls
- **setting**: Device selection, virtual background, quality stats
- **caption**: Live captions and translation
- **subsession**: Breakout rooms
- **recording**: Cloud recording
- **preview**: Pre-join device/background selection

### Context Providers
- **client-context.ts**: Provides Zoom Video SDK client throughout component tree
- **stream-context.ts**: Media stream context
- **session-additional-context.ts**: Additional session configuration

### Event System
`src/events/event-bus.ts` provides internal event pub/sub using the `mitt` library. Events defined in `src/events/event-constant.ts` include:
- Session lifecycle events (joined, closed, destroyed)
- Component visibility events
- Custom UI events

External SDK events are proxied through the UIToolkit's `on()`/`off()` methods.

### Build Configuration

**Vite** is used for both development and production builds (vite.config.ts):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zoom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

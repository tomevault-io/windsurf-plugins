---
trigger: always_on
description: description: Frontend Guideline
---

---
description: Frontend Guideline
globs: 
---
# DevHangout: Frontend Development Guidelines

## 1. Architecture Overview

DevHangout's frontend is a real-time, interactive application focused on delivering a seamless spatial experience for developers. The architecture follows a component-based approach with specialized systems for rendering, interaction, and communication.

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Application Shell                       │
├─────────────┬─────────────────────────┬─────────────────┤
│ Auth System │      Routing System     │   UI Library    │
├─────────────┴─────────────────────────┴─────────────────┤
│                      Core Systems                        │
├───────────┬──────────┬───────────┬──────────┬───────────┤
│ Rendering │ Movement │ Collision │  Input   │ Animation │
├───────────┴──────────┼───────────┼──────────┴───────────┤
│ Audio/Voice System   │ Network   │ Integration System   │
├────────────────────┬─┴───────────┴─┬────────────────────┤
│  Component Library │  State System  │    Asset System   │
└────────────────────┴────────────────┴────────────────────┘
```

### 1.2 Core Frontend Systems

1. **Rendering System** - Canvas/WebGL rendering for the virtual space
2. **Movement System** - Character movement and physics
3. **Input System** - Keyboard, mouse, and touch input handling
4. **Audio System** - WebRTC integration and spatial audio
5. **Network System** - WebSocket communication and state synchronization
6. **UI System** - Component library and overlay UI
7. **Asset System** - Loading and managing graphical assets

## 2. Technology Stack

### 2.1 Primary Technologies

| Component | Technology |
|-----------|------------|
| Framework | React |
| State Management | Redux Toolkit |
| Rendering | PixiJS (for 2D retro rendering) |
| Styling | CSS Modules with custom retro theming |
| UI Components | Custom 8-bit inspired component library |
| Networking | Socket.io client |
| Audio/Video | WebRTC with simple-peer |
| Build System | Vite |
| Testing | Vitest, React Testing Library |
| Code Quality | ESLint, Prettier |

### 2.2 Justification

- **React**: Provides component-based architecture and efficient rendering
- **Redux Toolkit**: For global state management with built-in immutability
- **PixiJS**: High-performance 2D WebGL renderer with good TypeScript support
- **Tailwind CSS**: Utility-first CSS for rapid UI development
- **Socket.io**: Reliable WebSocket implementation with fallbacks
- **Vite**: Fast development and optimized production builds

## 3. Project Structure

### 3.1 Directory Structure

```
src/
├── assets/            # Static assets (images, fonts, etc.)
├── components/        # Reusable React components
│   ├── ui/            # Basic UI components
│   ├── avatar/        # Avatar-related components
│   ├── space/         # Virtual space components
│   └── integration/   # Integration-related components
├── systems/           # Core functionality modules
│   ├── rendering/     # Canvas/WebGL rendering logic
│   ├── movement/      # Character movement and physics
│   ├── audio/         # Voice chat and spatial audio
│   ├── network/       # WebSocket and API communication
│   └── interaction/   # Object interaction logic
├── state/             # Redux state management
│   ├── slices/        # Redux Toolkit slices
│   ├── selectors/     # Memoized selectors
│   └── middleware/    # Custom Redux middleware
├── hooks/             # Custom React hooks
├── utils/             # Utility functions
├── pages/             # Full page components
├── routes/            # Routing configuration
├── api/               # API client functions
├── types/             # TypeScript type definitions
├── config/            # Configuration files
├── styles/            # Global styles and Tailwind config
└── App.tsx            # Application entry point
```

### 3.2 Component Structure

Components should follow a consistent directory structure:

```
ComponentName/
├── index.ts           # Exports the component
├── ComponentName.tsx  # Main component file
├── ComponentName.test.tsx  # Tests
├── ComponentName.module.css  # Component-specific styles (if needed)
└── hooks/             # Component-specific hooks
```

## 4. Rendering Architecture

### 4.1 Canvas Rendering

DevHangout uses a layered canvas approach:

```
┌─────────────────────────────────────────────┐
│               Overlay UI Layer               │ React DOM
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│               Interactive UI Layer           │ PixiJS
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│               Characters Layer               │ PixiJS
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│            Interactive Objects Layer         │ PixiJS
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│               Background Layer               │ PixiJS
└─────────────────────────────────────────────┘
```

### 4.2 Rendering Loop

The rendering system should implement:

1. A fixed timestep for physics and movement (60 FPS)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [virtualdev-xyz/dev-hangout-client](https://github.com/virtualdev-xyz/dev-hangout-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

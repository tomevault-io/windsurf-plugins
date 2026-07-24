---
trigger: always_on
description: > **AI Coding Agent Guidelines for Nora Music Player**
---

# Nora - GitHub Copilot Instructions

> **AI Coding Agent Guidelines for Nora Music Player**  
> Generated to help AI agents understand architectural patterns, conventions, and development workflows.

---

## 🎯 Project Overview

**Nora** is an elegant, feature-rich music player built with Electron and React, inspired by Oto Music (Android). It
emphasizes simplicity, beautiful design, and essential music management features that default music apps often lack.

### Core Technologies

- **Runtime**: Electron v39+ (main + renderer processes)
- **UI Framework**: React 19 with TypeScript (strict mode enabled)
- **Build System**: Vite + esbuild (electron-vite configuration)
- **State Management**: @tanstack/react-store with custom dispatch/store pattern
- **Data Fetching**: @tanstack/react-query with suspense queries
- **Routing**: TanStack Router
- **Database**: Drizzle ORM with PGlite (local PostgreSQL)
- **Styling**: Tailwind CSS v4 with dark mode support
- **Internationalization**: react-i18next
- **Testing**: Vitest with coverage reporting

### Key Features

- Organize songs, artists, albums, and playlists
- Synced and unsynced lyrics support
- Media Session API integration
- Discord Rich Presence integration
- Last.fm scrobbling support
- Custom metadata editing (MP3 only via node-id3)
- Dynamic theme generation from album artwork
- Mini-player mode with compact controls
- Global keyboard shortcuts

---

## 📁 Project Structure

```
nora/
├── src/
│   ├── main/                    # Electron main process
│   │   ├── main.ts             # Entry point: window management, IPC setup
│   │   ├── ipc.ts              # IPC handler registration
│   │   ├── db/                 # Database layer (Drizzle ORM + PGlite)
│   │   ├── core/               # Core business logic (library, playlists, etc.)
│   │   ├── fs/                 # File system watchers and operations
│   │   ├── auth/               # Last.fm authentication
│   │   └── other/              # Artworks, Discord RPC, utilities
│   │
│   ├── preload/                # Electron preload scripts
│   │   └── index.ts            # window.api bridge (IPC interface)
│   │
│   ├── renderer/               # React application
│   │   ├── src/
│   │   │   ├── App.tsx         # Main app component (365 lines, down from 2,013)
│   │   │   ├── hooks/          # 25+ custom React hooks for feature isolation
│   │   │   ├── store/          # TanStack Store configuration
│   │   │   ├── routes/         # TanStack Router routes
│   │   │   ├── components/     # React components
│   │   │   ├── other/          # Singleton services (AudioPlayer, PlayerQueue, etc.)
│   │   │   └── utils/          # Helper functions
│   │   └── index.html
│   │
│   ├── common/                 # Shared utilities (main + renderer)
│   │   ├── convert.ts
│   │   ├── isLyricsSynced.ts
│   │   ├── parseLyrics.ts
│   │   └── roundTo.ts
│   │
│   └── types/                  # TypeScript type definitions
│       ├── app.d.ts            # Core app types
│       └── [api].d.ts          # External API types
│
├── resources/                  # Static assets (icons, SQL migrations)
├── build/                      # Build artifacts and installer assets
└── test/                       # Vitest test files
```

---

## 🖥️ Main Process Architecture (Electron)

The main process is the heart of Nora's Electron application, handling system-level operations, database management,
file system watching, and IPC communication.

### Main Process Entry Point (`src/main/main.ts`)

**Responsibilities**:

- Window lifecycle management (create, resize, position, state)
- Player type switching (normal, mini, full-screen)
- System integration (tray, taskbar, global shortcuts)
- App lifecycle events (startup, quit, before-quit cleanup)
- Power management (prevent sleep, battery detection)
- System theme watching
- Single instance lock enforcement
- Protocol handling (`nora://` custom protocol)
- Auto-launch configuration

**Key Variables** (module-level state):

```typescript
export let mainWindow: BrowserWindow; // Main window instance
let tray: Tray; // System tray icon
let playerType: PlayerTypes; // 'normal' | 'mini' | 'full'
let isAudioPlaying: boolean; // Playback state for taskbar buttons
let currentSongPath: string; // For lyrics/metadata persistence
let powerSaveBlockerId: number | null; // Prevent display sleep during playback
```

**Window Size Constants**:

```typescript
// Normal window
MAIN_WINDOW_DEFAULT_SIZE_X = 1280
MAIN_WINDOW_DEFAULT_SIZE_Y = 720
MAIN_WINDOW_MIN_SIZE = 700x500

// Mini player
MINI_PLAYER_MIN_SIZE = 270x200
MINI_PLAYER_MAX_SIZE = 510x300
MINI_PLAYER_ASPECT_RATIO = 17/10
```

**Critical Functions**:

- `createWindow()`: Initialize BrowserWindow with preload script, frame settings, visual effects
- `manageWindowFinishLoad()`: Restore window position/size from settings, show window
- `handleBeforeQuit()`: Cleanup operations (save lyrics, metadata, close watchers, clear temp files)
- `changePlayerType(type)`: Switch between normal/mini/full-screen modes with size/position restoration
- `dataUpdateEvent(dataType, data, message)`: Debounced event aggregation for library updates

**System Integration**:

- **Single Instance**: Uses `app.requestSingleInstanceLock()` to prevent multiple instances

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sandakan/Nora](https://github.com/Sandakan/Nora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

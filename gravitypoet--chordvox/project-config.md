---
trigger: always_on
description: This document provides comprehensive technical details about the OpenWhispr project architecture for AI assistants working on the codebase.
---

# OpenWhispr Technical Reference for AI Assistants

This document provides comprehensive technical details about the OpenWhispr project architecture for AI assistants working on the codebase.

## Project Overview

OpenWhispr is an Electron-based desktop dictation application that uses whisper.cpp for speech-to-text transcription. It supports both local (privacy-focused) and cloud (OpenAI API) processing modes.

## Architecture Overview

### Core Technologies
- **Frontend**: React 19, TypeScript, Tailwind CSS v4, Vite
- **Desktop Framework**: Electron 36 with context isolation
- **Database**: better-sqlite3 for local transcription history
- **UI Components**: shadcn/ui with Radix primitives
- **Speech Processing**: whisper.cpp + NVIDIA Parakeet (via sherpa-onnx) + OpenAI API
- **Audio Processing**: FFmpeg (bundled via ffmpeg-static)

### Key Architectural Decisions

1. **Dual Window Architecture**:
   - Main Window: Minimal overlay for dictation (draggable, always on top)
   - Control Panel: Full settings interface (normal window)
   - Both use same React codebase with URL-based routing

2. **Process Separation**:
   - Main Process: Electron main, IPC handlers, database operations
   - Renderer Process: React app with context isolation
   - Preload Script: Secure bridge between processes

3. **Audio Pipeline**:
   - MediaRecorder API → Blob → ArrayBuffer → IPC → File → whisper.cpp
   - Automatic cleanup of temporary files after processing

## File Structure and Responsibilities

### Main Process Files

- **main.js**: Application entry point, initializes all managers
- **preload.js**: Exposes safe IPC methods to renderer via window.api

### Native Resources (resources/)

- **windows-key-listener.c**: C source for Windows low-level keyboard hook (Push-to-Talk)
- **globe-listener.swift**: Swift source for macOS Globe/Fn key detection
- **bin/**: Directory for compiled native binaries (whisper-cpp, nircmd, key listeners)

### Helper Modules (src/helpers/)

- **audioManager.js**: Handles audio device management
- **clipboard.js**: Cross-platform clipboard operations
  - macOS: AppleScript-based paste with accessibility permission check
  - Windows: PowerShell SendKeys with nircmd.exe fallback
  - Linux: Native XTest binary + compositor-aware fallbacks (xdotool, wtype, ydotool)
- **database.js**: SQLite operations for transcription history
- **debugLogger.js**: Debug logging system with file output
- **devServerManager.js**: Vite dev server integration
- **dragManager.js**: Window dragging functionality
- **environment.js**: Environment variable and OpenAI API management
- **hotkeyManager.js**: Global hotkey registration and management
  - Handles platform-specific defaults (GLOBE on macOS, backtick on Windows/Linux)
  - Auto-fallback to F8/F9 if default hotkey is unavailable
  - Notifies renderer via IPC when hotkey registration fails
  - Integrates with GnomeShortcutManager for GNOME Wayland support
- **gnomeShortcut.js**: GNOME Wayland global shortcut integration
  - Uses D-Bus service to receive hotkey toggle commands
  - Registers shortcuts via gsettings (visible in GNOME Settings → Keyboard → Shortcuts)
  - Converts Electron hotkey format to GNOME keysym format
  - Only active on Linux + Wayland + GNOME desktop
- **ipcHandlers.js**: Centralized IPC handler registration
- **windowsKeyManager.js**: Windows Push-to-Talk support with native key listener
  - Spawns native `windows-key-listener.exe` binary for low-level keyboard hooks
  - Supports compound hotkeys (e.g., `Ctrl+Shift+F11`, `CommandOrControl+Space`)
  - Emits `key-down` and `key-up` events for push-to-talk functionality
  - Graceful fallback if binary unavailable
- **menuManager.js**: Application menu management
- **tray.js**: System tray icon and menu
- **whisper.js**: Local whisper.cpp integration and model management
- **parakeet.js**: NVIDIA Parakeet model management via sherpa-onnx
- **parakeetServer.js**: sherpa-onnx CLI wrapper for transcription
- **windowConfig.js**: Centralized window configuration
- **windowManager.js**: Window creation and lifecycle management

### React Components (src/components/)

- **App.jsx**: Main dictation interface with recording states
- **ControlPanel.tsx**: Settings, history, model management UI
- **OnboardingFlow.tsx**: 8-step first-time setup wizard
- **SettingsPage.tsx**: Comprehensive settings interface
- **WhisperModelPicker.tsx**: Model selection and download UI
- **ui/**: Reusable UI components (buttons, cards, inputs, etc.)

### React Hooks (src/hooks/)

- **useAudioRecording.js**: MediaRecorder API wrapper with error handling
- **useClipboard.ts**: Clipboard operations hook
- **useDialogs.ts**: Electron dialog integration
- **useHotkey.js**: Hotkey state management
- **useLocalStorage.ts**: Type-safe localStorage wrapper
- **usePermissions.ts**: System permission checks and settings access
  - `openMicPrivacySettings()`: Opens OS microphone privacy settings
  - `openSoundInputSettings()`: Opens OS sound input device settings
  - `openAccessibilitySettings()`: Opens OS accessibility settings (macOS only)
- **useSettings.ts**: Application settings management
- **useWhisper.ts**: Whisper binary availability check


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GravityPoet/ChordVox](https://github.com/GravityPoet/ChordVox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

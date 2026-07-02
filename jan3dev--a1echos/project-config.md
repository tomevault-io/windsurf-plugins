---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Echos is a React Native voice notes app with on-device transcription built using Expo. It records audio and transcribes it locally using ASR models, supporting both real-time streaming transcription and file-based transcription modes.

## Development Commands

### Setup & Running

```bash
npm install           # Install dependencies
npm start             # Start Expo dev server
npm run ios           # Run iOS development build (recommended)
npm run android       # Run Android development build (recommended)
```

**Important**: This app uses native modules and **requires a development build**. Standard Expo Go will not work.

### Storybook

```bash
EXPO_PUBLIC_STORYBOOK_ENABLED=true npm start
```

### Linting

```bash
npm run lint          # Run ESLint
```

### Testing

```bash
npm run test              # Run all tests
npm run test:watch        # Run tests in watch mode
npm run test:coverage     # Run tests with coverage report
```

**Coverage Thresholds** (enforced in `jest.config.js`):

| Metric     | Minimum |
| ---------- | ------- |
| Statements | 95%     |
| Branches   | 90%     |
| Functions  | 95%     |
| Lines      | 95%     |

Always run `npm run test:coverage` after changes to verify thresholds are met.

## Architecture

### Core Technology Stack

- **Framework**: Expo SDK 55 + React Native 0.83+ + React 19.2
- **Navigation**: expo-router (file-based routing)
- **State Management**: Zustand
- **Localization**: i18next + react-i18next
- **Audio**: expo-audio
- **Transcription**: react-native-sherpa-onnx (on-device Whisper + NeMo Parakeet inference)
- **Storage**: expo-file-system + expo-secure-store
- **Encryption**: expo-crypto + react-native-aes-gcm-crypto

### Path Aliases

The project uses TypeScript path aliases (configured in tsconfig.json):

```typescript
@/*                    // Root
@/components/*         // UI components
@/services/*           // Business logic services
@/stores/*             // Zustand stores
@/models/*             // TypeScript types/interfaces
@/assets/*             // Static assets
@/utils/*              // Utility functions
@/constants/*          // App constants
@/theme/*              // Theme system
@/hooks/*              // React hooks
@/localization/*       // i18n translations
```

### Directory Structure

#### `/app` - Expo Router Pages

- `(pages)/` - Main app screens (home, session detail)
- `(storybook)/` - Storybook UI dev environment
- `_layout.tsx` - Root layout with app initialization, theme, global components

#### `/components` - Three-tier Component Architecture

1. **`domain/`** - Feature-specific components (home, session, settings, transcription)
2. **`shared/`** - Cross-feature reusable components (error-view, list-item, recording-controls)
3. **`ui/`** - Primitive UI components (button, checkbox, modal, text, icon, etc.)

#### `/services` - Core Business Logic

All services follow singleton pattern and are exported from `services/index.ts`:

- **AudioSessionService** - iOS AVAudioSession configuration management
- **SherpaTranscriptionService** - sherpa-onnx STT engine init, real-time + file transcription (replaced WhisperService)
- **ModelDownloadService** - Downloads non-bundled models from HuggingFace via `expo-file-system/legacy` `createDownloadResumable` (streams to disk, progress callbacks, cancel support)
- **BackgroundRecordingService** - Android foreground service for background recording
- **StorageService** - Session/transcription CRUD with encrypted file storage
- **EncryptionService** - AES-GCM encryption for audio files
- **PermissionService** - Audio recording permission management
- **ShareService** - System share sheet integration

#### `/stores` - Zustand State Management

Five main stores (all exported from `stores/index.ts`):

1. **sessionStore** - Session CRUD, active session tracking, incognito mode
2. **transcriptionStore** - Recording state machine, transcription CRUD, sherpa-onnx coordination
3. **settingsStore** - User preferences (theme, language, model, incognito mode). Auto-resets language to English when switching to a model that doesn't support the current language.
4. **uiStore** - UI state (tooltips, toasts, selection modes, recording controls visibility)
5. **modelDownloadStore** - Download progress tracking, downloaded model verification

**Critical**: Stores have initialization functions that must be called in order:

```typescript
await initializeSettingsStore();
await initializeSessionStore();
await initializeTranscriptionStore(); // Depends on sessionStore
```

#### `/models` - TypeScript Types

Domain models with JSON serialization helpers:

- **Session** - Recording session with metadata
- **Transcription** - Audio + transcript with timing
- **TranscriptionState** - State machine states (IDLE, RECORDING, TRANSCRIBING, etc.)
- **AppTheme** - Theme enum (LIGHT, DARK, SYSTEM)
- **ModelType** - Whisper model variants

#### `/theme` - Design System

- **themeColors.ts** - Light/dark theme color tokens
- **typography.ts** - Font definitions (Manrope, PublicSans)
- **shadows.ts** - iOS/Android platform-specific shadows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jan3dev/a1echos](https://github.com/jan3dev/a1echos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->

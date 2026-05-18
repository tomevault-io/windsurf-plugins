---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Worktrees

Worktree directory: `.claude/worktrees/` (gitignored)

## Project Overview

Sokuji is a real-time AI-powered translation application available as both an Electron desktop app and a browser extension. It provides live speech translation using OpenAI, Google Gemini, Palabra.ai, and Kizuna AI APIs with modern audio processing capabilities. It also supports OpenAI-compatible API endpoints for flexibility.

## Development Commands

### Running the Application
```bash
# Run Electron app in development mode
npm run electron:dev

# Run React app only (for browser extension development)
npm run dev

# Build Electron app for production
npm run electron:build

# Run tests
npm run test

# Run tests with UI
npm run test:ui

# Run specific test
npm run test -- path/to/test
```

### Building and Packaging
```bash
# Build React app
npm run build

# Package Electron app
npm run package

# Create distributable packages
npm run make
```

### Version Update Process

**All five version sites must land in a single `chore(release): vX.Y.Z` commit BEFORE the tag is created.** Earlier releases split root and extension version bumps into two separate commits with the tag on the root-only commit; the tag checkout then built the extension with the previous version. The release workflow checks out the tag verbatim, so every version-affecting file must be at the new version at the tagged commit.

1. Update all five files in one go:
   - `package.json`
   - `extension/package.json`
   - `extension/manifest.json`
   - `package-lock.json` (run `npm install` at the root to regenerate)
   - `extension/package-lock.json` (run `npm install` inside `extension/` to regenerate)
2. Commit all five together: `git commit -m "chore(release): vX.Y.Z"`
3. Create annotated tag on that commit: `git tag -a vX.Y.Z -m "Release vX.Y.Z"`
4. Push: `git push origin main --follow-tags`

**Sanity check before pushing the tag:**

```bash
git show vX.Y.Z:package.json          | grep '"version"'
git show vX.Y.Z:extension/package.json | grep '"version"'
git show vX.Y.Z:extension/manifest.json | grep '"version"'
```

All three must print the same new version. If they don't, `git tag -d vX.Y.Z` and re-tag on the correct commit before pushing.

## Architecture Overview

### Dual Platform Architecture
The codebase supports both Electron desktop app and Chrome/Edge browser extension from a shared React codebase:
- **Shared code**: `src/` directory contains all React components and business logic
- **Electron-specific**: `electron/` directory, virtual audio device management (Linux only)
- **Extension-specific**: `extension/` directory, manifest.json, background scripts

### Key Architectural Components

1. **Service Layer Pattern**
   - `ServiceFactory` creates platform-specific implementations with singleton caching
   - All services implement interfaces (IAudioService, ISettingsService)
   - Platform detection via `src/utils/environment.ts` utilities

2. **AI Client Architecture**
   - `ClientFactory` creates provider-specific clients
   - Providers: OpenAI, Gemini, PalabraAI, KizunaAI, OpenAI Compatible
   - Each client implements `IClient` interface
   - Real-time communication via WebSocket or REST APIs
   - OpenAI Compatible provider allows custom API endpoints (Electron only)
   - KizunaAI uses OpenAI-compatible API with backend-managed authentication

3. **Audio Processing Pipeline**
   ```
   Input Device → ModernAudioRecorder → AI Provider → ModernAudioPlayer → Output Device
   ```
   - `ModernAudioRecorder`: Captures input with echo cancellation, supports AudioWorklet with ScriptProcessor fallback
   - `ModernAudioPlayer`: Queue-based playback with event-driven processing and volume control
   - Unified audio service across all platforms with virtual device support in Electron (Linux only)

4. **State Management**
   - **Zustand stores** in `src/stores/` for primary application state:
     - `settingsStore.ts`: Provider settings, API keys, validation state, UI mode
     - `sessionStore.ts`: Active session state and conversation items
     - `audioStore.ts`: Audio device selection and playback state
     - `logStore.ts`: Application logs and diagnostics
   - React Context for specific features: OnboardingContext, UserProfileContext
   - Zustand's `subscribeWithSelector` middleware for efficient re-renders
   - Backend-managed API key integration for authenticated providers

5. **Audio Service Management**
   - `ModernBrowserAudioService` provides unified audio handling
   - Cross-platform compatibility without virtual devices
   - Automatic device switching and reconnection, including dynamic switching during active sessions

## Important Patterns and Conventions

### Code Organization
- `src/components/` - Functional React components with TypeScript
- `src/stores/` - Zustand state management stores
- `src/services/` - Service layer with interface contracts
- `src/services/clients/` - AI provider client implementations
- `src/services/providers/` - Provider-specific configurations
- `src/lib/modern-audio/` - Web Audio API modules (JavaScript, not TypeScript)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kizuna-ai-lab/sokuji](https://github.com/kizuna-ai-lab/sokuji) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->

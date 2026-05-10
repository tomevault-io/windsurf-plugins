---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Development
- `npm run lint-fix` - Automatically fix linting issues
- `npm run tsc:ci` - Run TypeScript checks for both web and native platforms
- `npm run tsc:web` - TypeScript check for web platform only
- `npm run tsc:native` - TypeScript check for native platform only
- `npm run lint:ci` - Run ESLint without type checking
- `make dev` - Start development server with webpack-dev-server
- `make compile` - Build production bundles
- `make clean` - Clean build directory
- `make all` - Full build (compile + deploy)

### Testing
- `npm test` - Run full test suite using WebDriverIO
- `npm run test-single -- <spec-file>` - Run single test file
- `npm run test-dev` - Run tests against development environment
- `npm run test-dev-single -- <spec-file>` - Run single test in dev mode


### Language Tools
- `npm run lang-sort` - Sort language files
- `npm run lint:lang` - Validate JSON language files

### Platform-Specific TypeScript
TypeScript configuration is split between web and native platforms with separate tsconfig files.

## Architecture Overview

### Multi-Platform Structure
Jitsi Meet supports both web and React Native platforms with platform-specific file extensions and directories:
- `.web.ts/.web.tsx` - Web-specific implementations
- `.native.ts/.native.tsx` - React Native-specific implementations
- `.any.ts/.any.tsx` - Shared cross-platform code
- `.android.ts/.android.tsx` - Android-specific code
- `.ios.ts/.ios.tsx` - iOS-specific code
- `web/` directories - Web-specific components and modules
- `native/` directories - React Native-specific components and modules
- `react/features/mobile/` - Native-only features

### Core Directories
- `react/features/` - Main application features organized by domain (83+ feature modules)
- `modules/` - Legacy JavaScript modules and APIs
- `css/` - SCSS stylesheets compiled to CSS
- `libs/` - Compiled output directory for JavaScript bundles
- `static/` - Static assets and HTML files
- `tests/` - WebDriverIO end-to-end tests

### Feature-Driven Architecture
The application is organized under `react/features/` with each feature containing:

- **`actionTypes.ts`** - Redux action type constants
- **`actions.ts`** - Redux action creators (platform-specific variants with `.any.ts`, `.web.ts`, `.native.ts`)
- **`reducer.ts`** - Redux reducer functions
- **`middleware.ts`** - Redux middleware for side effects
- **`functions.ts`** - Utility functions and selectors
- **`constants.ts`** - Feature-specific constants
- **`logger.ts`** - Feature-specific logger instance
- **`types.ts`** - TypeScript type definitions

### Key Application Files
- `app.js` - Main web application entry point
- `webpack.config.js` - Multi-bundle Webpack configuration
- `Makefile` - Build system for development and production
- `package.json` - Dependencies and scripts with version requirements

### Bundle Architecture
The application builds multiple bundles:
- `app.bundle.js` / `app.bundle.min.js` - Main application bundle (entry: `./app.js`)
- `external_api.js` / `external_api.min.js` - External API for embedders (entry: `./modules/API/external/index.js`)
- `alwaysontop.js` / `alwaysontop.min.js` - Always-on-top window functionality (entry: `./react/features/always-on-top/index.tsx`)
- `close3.js` / `close3.min.js` - Close3 functionality (entry: `./static/close3.js`)
- `face-landmarks-worker.js` / `face-landmarks-worker.min.js` - Face landmarks detection worker (entry: `./react/features/face-landmarks/faceLandmarksWorker.ts`)
- `noise-suppressor-worklet.js` / `noise-suppressor-worklet.min.js` - Audio noise suppression worklet (entry: `./react/features/stream-effects/noise-suppression/NoiseSuppressorWorklet.ts`)
- `screenshot-capture-worker.js` / `screenshot-capture-worker.min.js` - Screenshot capture worker (entry: `./react/features/screenshot-capture/worker.ts`)

### Redux Architecture
Features follow a Redux-based architecture with:
- Actions, reducers, and middleware in each feature directory
- Cross-platform state management
- Modular feature organization with clear boundaries

The codebase uses a registry-based Redux architecture:
- **ReducerRegistry** - Features register their reducers independently
- **MiddlewareRegistry** - Features register middleware without cross-dependencies
- **IReduxState** - Global state is strongly typed with 80+ feature states

### Dependencies
- Uses `lib-jitsi-meet` as the core WebRTC library
- React with TypeScript support
- React Native for mobile applications
- Webpack for bundling with development server

### TypeScript Configuration
- `tsconfig.web.json` - Web platform TypeScript config (excludes native files)
- `tsconfig.native.json` - React Native TypeScript config (excludes web files)
- Strict TypeScript settings with ES2024 target
- Platform-specific module suffixes (`.web`, `.native`)

### Key Base Features
- **`base/app/`** - Application lifecycle management
- **`base/conference/`** - Core conference logic
- **`base/tracks/`** - Media track management
- **`base/participants/`** - Participant management
- **`base/config/`** - Configuration management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omarnebi/JitsiMeet](https://github.com/omarnebi/JitsiMeet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->

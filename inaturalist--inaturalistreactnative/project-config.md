---
trigger: always_on
description: This file provides guidance for AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working with code in this repository.

## Project Overview

iNaturalistReactNative is the official iNaturalist mobile client written in React Native, replacing the legacy iOS and Android native apps. It's a community science platform where users photograph organisms, upload observations, and get AI-powered species identifications.

## Development Commands

### Running the App
```bash
# Start Metro bundler (with cache reset recommended to avoid build issues)
npm start -- --reset-cache

# Run on iOS
npm run ios

# Run on Android
npm run android

# Run release builds
npm run ios:release
npm run android:release
```

### Testing
```bash
# Run all tests (integration + unit)
npm test

# Run only unit tests
npm run test:unit

# Run only integration tests
npm run test:integration

# Run individual test by name
npx jest

# E2E tests (requires Detox setup)
npm run e2e              # Build and test both iOS + Android (use e2e:ios for iOS-only)
npm run e2e:android      # Build and test Android
npm run e2e:test         # Run tests without rebuilding
```

### Linting
```bash
# Run all linters (eslint, flow, rubocop)
npm run lint

# Auto-fix linting issues
npm run lint:fix

# Run individual linters
npm run lint:eslint
npm run lint:flow       # Flow type checking
npm run lint:rubocop    # Ruby linting (fastlane)
npm run lint:tsc        # TypeScript checking
```

**Note on `lint:tsc`:** The repo has a large pre-existing TypeScript error baseline (over 1,000 errors), so a clean run is not expected. The standard is to add no new errors in files you touch — verify by filtering the tsc output for your filenames and comparing against the pre-change state. TS7016 implicit-`any` errors from importing untyped `.js` modules are part of the accepted baseline; do not add `@ts-ignore` comments for them.

**Never silence a type error you can't honestly fix.** Casts (`as unknown as X`), non-null `!`, and widening a type to `object`/`Function` quiet `tsc` while leaving the mismatch in shipped code. If an error is a legitimate mismatch that can't be resolved within the scope of your change, leave it reported

### Translations
```bash
# Build translation JSON from Fluent files
npm run translate

# Prepare fastlane metadata for app stores
npm run prepare-fastlane-metadata
```

### Icons & Assets
```bash
# Rebuild icon font from SVG files in src/images/icons/
npm run icons

# Add example AI model files
npm run add-example-model
```

### Cleanup
```bash
# Clean project (interactive menu for caches, builds, pods, node_modules)
npx react-native clean-project

# Full clean and restart
npm run clean-start
```

## Code Architecture

### Navigation Structure

The app uses React Navigation 7 with `@react-navigation/native-stack` and `@react-navigation/bottom-tabs` in a nested hierarchy:

1. **RootStackNavigator** (NativeStack, top level) - `src/navigation/RootStackNavigator.tsx`
   - **OnboardingStackNavigator** - Rendered in place of the tab navigator when `!onboardingShown` (not a modal over it)
   - **BottomTabNavigator** - Shown when `onboardingShown`. Contains four tabs:
     - `MenuTab` → TabStackNavigator (initialRouteName: "Menu")
     - `ExploreTab` → TabStackNavigator (initialRouteName: "RootExplore")
     - `ObservationsTab` → TabStackNavigator (initialRouteName: "ObsList")
     - `NotificationsTab` → TabStackNavigator (initialRouteName: "Notifications")
   - **NoBottomTabStackNavigator** - Camera, PhotoLibrary, GroupPhotos, SoundRecorder, plus SharedStackScreens
   - **LoginStackNavigator** - Login, SignUp, ForgotPassword, etc.

Key design patterns:
- All four bottom tabs share the **same `TabStackNavigator` component** (`src/navigation/StackNavigators/TabStackNavigator.tsx`) with different `initialRouteName` values, giving every tab access to the full screen catalog.
- **`SharedStackScreens`** (`src/navigation/StackNavigators/SharedStackScreens.tsx`) is a `Stack.Group` rendered in both `TabStackNavigator` and `NoBottomTabStackNavigator`, so screens like ObsEdit, TaxonDetails, Match, and Suggestions work from either context.
- The `NavigationContainer` lives in `src/navigation/OfflineNavigationGuard.tsx` with a global `navigationRef` from `src/navigation/navigationUtils.ts`.
- Deep linking is handled manually via React Native's `Linking` API in `src/components/hooks/useLinking.ts` (no React Navigation `linking` config).

To add a new screen, place it in the appropriate navigator: `TabStackNavigator` if it needs bottom tabs visible, `NoBottomTabStackNavigator` if not, or `SharedStackScreens` if it needs to be reachable from both contexts.

**Details:** the full navigator hierarchy, screen-param typing, and deep-linking are in `agent-docs/architecture/navigation-patterns.md`.

### State Management

The app uses a hybrid state management approach:

- **Zustand stores** (`src/stores/`) - Global app state slices. Key examples include:
  - `createObservationFlowSlice` - Observation creation/editing flow
  - `createUploadObservationsSlice` - Upload queue and status
  - `createSyncObservationsSlice` - Syncing observations from server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inaturalist/iNaturalistReactNative](https://github.com/inaturalist/iNaturalistReactNative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

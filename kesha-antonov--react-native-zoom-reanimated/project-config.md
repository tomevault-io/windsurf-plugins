---
trigger: always_on
description: React Native library providing a zoomable view component built with react-native-reanimated and react-native-gesture-handler. The library includes a main source component and a complete React Native example application.
---

# react-native-zoom-reanimated Development Guide

React Native library providing a zoomable view component built with react-native-reanimated and react-native-gesture-handler. The library includes a main source component and a complete React Native example application.

**Always reference these instructions first and only fall back to search or bash commands when you encounter unexpected information that differs from the info documented here.**

## Package Manager

**IMPORTANT: This project uses Yarn as the package manager. Always use `yarn` instead of `npm` for all package operations.**

## Working Effectively

### Bootstrap and Environment Setup
- **Node.js requirement**: Project uses Node.js v20+. No additional Node.js installation needed.
- **Dependencies installation**:
  - Root library: `yarn install` -- takes 45-60 seconds. NEVER CANCEL.
  - Example app: `cd example && yarn install` -- takes 40-50 seconds. NEVER CANCEL.
- **Initial setup for development**:
  ```bash
  # Install root dependencies
  yarn install

  # Install example dependencies
  cd example && yarn install
  ```

### Development and Build Commands

**Library Development:**
- **TypeScript compilation**: `yarn tsc --noEmit` -- takes 1-2 seconds. Validates library source code.
- **Library linting**: `yarn eslint -c .eslintrc.js --ext .ts,.tsx src/` -- takes 1-2 seconds.
- **Library autofix**: `yarn eslint -c .eslintrc.js --ext .ts,.tsx --fix src/` -- automatically fixes linting issues in src/

**Example App Development:**
- **Start Metro bundler**: `cd example && yarn start` -- takes 15-20 seconds to start. NEVER CANCEL.
  - Shows Metro ASCII art logo when ready
  - Interactive menu: 'i' for iOS, 'a' for Android, 'd' for dev menu, 'r' for reload
  - Use Ctrl+C to stop
- **TypeScript validation**: `cd example && yarn tsc --noEmit` -- takes 2-3 seconds.
- **Linting**: `cd example && yarn lint` -- takes 1-2 seconds.
- **Lint autofix**: `cd example && yarn eslint . --fix` -- takes 1-2 seconds.

### Build Validation and Testing

**Bundle Creation (works without devices):**
- **Android bundle**: `cd example && yarn react-native bundle --platform android --dev false --entry-file index.js --bundle-output bundle.js --assets-dest /tmp/` -- takes 20-25 seconds. NEVER CANCEL.
- **iOS bundle**: `cd example && yarn react-native bundle --platform ios --dev false --entry-file index.js --bundle-output bundle-ios.js --assets-dest /tmp/` -- takes 20-25 seconds. NEVER CANCEL.

**Testing Limitations:**
- **Jest tests**: Root Jest tests fail due to configuration issues with TSX/JSX parsing. This is a known limitation.
- **Example tests**: `cd example && yarn test` fails because React Native native modules are not available in Jest (expected behavior for RN libraries).
- **Device testing**: Cannot run on physical devices or emulators in this environment. Use bundle creation to validate builds.

## Validation Requirements

**ALWAYS run these validation steps after making changes:**

1. **TypeScript validation**:
   ```bash
   # Library code
   yarn tsc --noEmit

   # Example app code
   cd example && yarn tsc --noEmit
   ```

2. **Linting validation and fixes**:
   ```bash
   # Fix library linting issues (note: autofix only handles style issues, not TypeScript errors)
   yarn eslint -c .eslintrc.js --ext .ts,.tsx --fix src/

   # Fix example linting issues
   cd example && yarn eslint . --fix

   # Verify linting status (may show remaining issues that need manual fixes)
   yarn eslint -c .eslintrc.js --ext .ts,.tsx src/
   cd example && yarn lint
   ```

3. **Bundle validation** (proves the app builds correctly):
   ```bash
   cd example
   # Test Android bundle
   yarn react-native bundle --platform android --dev false --entry-file index.js --bundle-output bundle.js --assets-dest /tmp/
   # Test iOS bundle
   yarn react-native bundle --platform ios --dev false --entry-file index.js --bundle-output bundle-ios.js --assets-dest /tmp/
   # Clean up
   rm bundle.js bundle-ios.js
   ```

4. **Metro bundler validation**:
   ```bash
   cd example
   # Start Metro (should show ASCII logo and "Dev server ready" message)
   yarn start
   # Stop with Ctrl+C after confirming it starts successfully
   ```

## Manual Validation Scenarios

**After making changes to the library, always test these scenarios:**

1. **Component Integration Test**:
   - Start Metro: `cd example && yarn start`
   - Verify the example app has toggle functionality between single image and gallery modes
   - Confirm both modes use the Zoom component correctly
   - Check that TypeScript compilation passes for all component usage patterns

2. **API Compatibility Test**:
   - Review that `src/index.tsx` exports match the usage in `example/App.tsx` and `example/FlatListExample.tsx`
   - Verify that prop interfaces and TypeScript types are consistent
   - Test that configuration props (doubleTapConfig, etc.) work as expected

3. **Build Integration Test**:
   - Create bundles for both platforms to ensure no runtime import errors
   - Validate that all required peer dependencies are properly referenced

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kesha-antonov/react-native-zoom-reanimated](https://github.com/kesha-antonov/react-native-zoom-reanimated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

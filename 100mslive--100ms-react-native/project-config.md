---
trigger: always_on
description: Project-specific guidance for Claude Code when working with the 100ms React Native SDK monorepo.
---

# CLAUDE.md

Project-specific guidance for Claude Code when working with the 100ms React Native SDK monorepo.

## Project Overview

- **Type**: React Native monorepo for video conferencing SDK
- **Main packages**:
  - `@100mslive/react-native-hms` - Core SDK with native modules (iOS/Android)
  - `@100mslive/react-native-room-kit` - Prebuilt UI components
  - `@100mslive/react-native-video-plugin` - Video player plugin
- **Sample apps**: HMSReactNativeSample, expo demo, callkeep demo, quickstart, virtual background
- **Package manager**: npm (NOT yarn)
- **Repository**: https://github.com/100mslive/100ms-react-native

## Development Environment

### Required Versions

- **Node.js**: v22.20.0 (ENFORCED via .nvmrc - always use `nvm use`)
- **npm**: v10.9.3 (comes with Node via nvm)
- **React Native**: 0.73.0+ (minimum), 0.77.3 (recommended)
- **React**: 18.2.0+
- **TypeScript**: 4.6.3 (react-native-hms), 5.0.2 (react-native-room-kit)
- **Java**: 17+ (specified in `.java-version` file)
- **Ruby**: For Fastlane and CocoaPods
- **iOS**: 16.0+ (minimum)
- **Android**: API 24+ (Android 7.0)

### Required Tools

- nvm (Node Version Manager)
- CocoaPods (`gem install cocoapods`)
- Fastlane (`gem install fastlane`)
- bundler (`gem install bundler`)
- Android SDK (for Android development)
- Xcode (for iOS development)

### Initial Setup

```bash
# Always use correct Node version
nvm use

# Install root dependencies
npm install

# For react-native-hms package
cd packages/react-native-hms
npm install
npm run prepack  # Builds the package

# For example apps
cd example
npm install
cd ios && pod install  # iOS only
```

## Monorepo Structure

```
100ms-react-native/
├── packages/
│   ├── react-native-hms/          # Core SDK with native modules
│   │   ├── src/                   # TypeScript source
│   │   ├── android/               # Android native code
│   │   ├── ios/                   # iOS native code
│   │   ├── lib/                   # Built output (gitignored)
│   │   └── example/               # Example app for testing
│   ├── react-native-room-kit/     # Prebuilt UI Kit
│   │   ├── src/                   # React components
│   │   └── example/               # Example app
│   └── react-native-video-plugin/ # Video player
├── sample-apps/                   # Standalone sample applications
├── scripts/                       # Build and utility scripts
├── .github/workflows/             # CI/CD configuration
└── release-apps.sh                # Release automation script
```

## Common Development Commands

### Package Development

```bash
# Build packages (required before testing changes)
cd packages/react-native-hms
npm run prepack  # Uses react-native-builder-bob

cd packages/react-native-room-kit
npm run prepack

# Lint code
npm run lint

# Type check
npm run typescript  # or npm run typecheck

# Run tests
npm test
```

### Example Apps

```bash
# Run Android
cd packages/react-native-hms/example
npx react-native run-android

# Run iOS
cd packages/react-native-hms/example
cd ios && pod install && cd ..
npx react-native run-ios
```

### Release Process

```bash
# Release sample apps (Android & iOS)
./release-apps.sh

# Options:
./release-apps.sh --android-only
./release-apps.sh --ios-only
./release-apps.sh --dry-run
./release-apps.sh --no-commit
```

## Code Style & Standards

### Prettier Configuration (MUST FOLLOW)

- Single quotes (not double)
- Tab width: 2 spaces
- Trailing commas: ES5 style
- No tabs (use spaces)
- Consistent quote props

### ESLint

- Extends: `@react-native-community`, `prettier`
- Pre-commit hook runs lint automatically
- Fix with: `npm run lint -- --fix`

### TypeScript

- Strict mode enabled
- Use proper types (no `any` unless necessary)
- Export types for public APIs

### Commit Messages (ENFORCED)

- Use Conventional Commits format
- Examples:
  - `feat: add screen sharing support`
  - `fix: resolve audio mute issue on Android`
  - `chore: update dependencies`
  - `docs: update installation guide`
- Pre-commit hooks run: `npm run lint && npm run typescript`

## Build & Release Process

### Package Building

- Uses `react-native-builder-bob` for compilation
- Outputs to `lib/` directory (commonjs, module, typescript)
- Always run `npm run prepack` after source changes
- Check `package.json` "files" field for published content

### Native Builds

**Android:**

- Gradle build system
- Build files: `android/build.gradle`, `android/app/build.gradle`
- Version bumping: Update `versionCode` and `versionName`

**iOS:**

- CocoaPods for dependency management
- Podspec file: `react-native-hms.podspec`
- Always run `pod install` after native changes
- Xcode project: `ios/RNExample.xcodeproj`

### Release Script

- `release-apps.sh` automates Android & iOS distribution
- Uses Fastlane lanes for build and deploy
- Automatically bumps versions
- Can commit changes or run dry-run

## Git Workflow & CI/CD

### Branches

- Main branch: `main`
- Feature branches: Create from `main`
- Always sync with remote before creating PRs

### Pre-commit Hooks

- Managed by Husky/Lefthook
- Automatically runs: lint, typescript check
- Must pass before commit is created

### CI/CD Workflows

- `build.yml` - Validates PR builds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [100mslive/100ms-react-native](https://github.com/100mslive/100ms-react-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

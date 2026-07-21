---
trigger: always_on
description: This document provides guidance for AI agents working on the Discourse Mobile codebase.
---

# AGENTS.md - Discourse Mobile Codebase Guide

This document provides guidance for AI agents working on the Discourse Mobile codebase.

## Project Overview

**Discourse Mobile** is a native iOS and Android application for [Discourse](https://discourse.org) forums, built with React Native. It allows users to connect to multiple Discourse communities, receive push notifications, browse topics, and authenticate via OAuth2.

### Technology Stack

- **Framework**: React Native
- **JS Engine**: Hermes
- **Navigation**: React Navigation v6 (Stack + Bottom Tabs)
- **State**: React hooks, Context (ThemeContext), AsyncStorage
- **Push Notifications**: Firebase Cloud Messaging (FCM)
- **Authentication**: OAuth2 via Safari Web Auth with RSA encryption
- **Testing**: Detox (e2e), Jest (unit)
- **Languages**: JavaScript, Swift (iOS Share Extension), Kotlin (Android)

## Project Structure

```
DiscourseMobile/
├── js/                          # Main JavaScript source
│   ├── Discourse.js             # Root app component, navigation, lifecycle
│   ├── site_manager.js          # Multi-site management, auth tokens
│   ├── site.js                  # Site model, API interactions
│   ├── DiscourseUtils.js        # Notification routing logic
│   ├── ThemeContext.js          # Dark/light theme configuration
│   ├── screens/                 # Screen components
│   │   ├── HomeScreen.js        # Main sites list, topic viewing
│   │   ├── NotificationsScreen.js
│   │   ├── DiscoverScreen.js    # Site discovery
│   │   ├── AddSiteScreen.js     # Connect to new sites
│   │   ├── SettingsScreen.js
│   │   ├── WebViewScreen.js     # In-app web content
│   │   └── *Components/         # Screen-specific sub-components
│   ├── platforms/               # Platform-specific implementations
│   │   ├── firebase.ios.js
│   │   ├── firebase.android.js
│   │   └── background-fetch.*.js
│   └── locale/                  # 49 language translation files (JSON)
├── lib/                         # Utility libraries
│   ├── fetch.js                 # Custom fetch wrapper
│   ├── jsencrypt.js             # RSA encryption
│   └── random-bytes.js          # CSPRNG utility
├── ios/                         # iOS native code
│   ├── Discourse/               # Main app target
│   ├── ShareExtension/          # iOS Share Extension (Swift)
│   └── Podfile                  # CocoaPods dependencies
├── android/                     # Android native code
│   └── app/src/main/java/com/discourse/
├── e2e/                         # Detox e2e tests
└── fastlane/                    # CI/CD automation
```

## Key Files

| File                       | Purpose                                                       |
| -------------------------- | ------------------------------------------------------------- |
| `js/Discourse.js`          | Root component, navigation setup, deep linking, auth handling |
| `js/site_manager.js`       | Manages connected sites, auth tokens, device registration     |
| `js/site.js`               | Site model class, API calls, basic info fetching              |
| `js/DiscourseUtils.js`     | Maps 37+ notification types to endpoints and icons            |
| `js/ThemeContext.js`       | Theme definitions (colors, fonts) for light/dark mode         |
| `js/screens/HomeScreen.js` | Main UI with draggable site list and topic viewing            |

## Architecture Patterns

### Platform-Specific Code

Use file suffixes for platform divergence:

- `*.ios.js` - iOS-specific implementation
- `*.android.js` - Android-specific implementation

The bundler automatically selects the correct file based on platform.

### Component Organization

- Screens in `js/screens/`
- Screen-specific components in `js/screens/{ScreenName}Components/`
- Shared components in `js/screens/CommonComponents/`

### State Management

- **Local state**: React `useState` hooks
- **App-wide theme**: `ThemeContext` (React Context)
- **Site data**: `SiteManager` singleton class
- **Persistence**: `AsyncStorage` for local storage

### Authentication Flow

1. User initiates OAuth in `AddSiteScreen`
2. `SiteManager` generates auth URL with state/challenge
3. Safari Web Auth opens Discourse authorization page
4. User approves, redirected to `discourse://auth_redirect`
5. App exchanges code for token using RSA encryption
6. Token stored in AsyncStorage

## Development Commands

```bash
# Install dependencies
yarn

# iOS setup
bundle install
cd ios && pod install && cd ..

# Start Metro bundler
npx react-native start

# Run on iOS
npx react-native run-ios

# Run on Android
npx react-native run-android

# Run e2e tests
npx detox build --configuration ios.sim.debug
npx detox test --configuration ios.sim.debug

# Lint
yarn lint
```

## Build Configuration

### iOS

- **Min Deployment**: iOS 15.1
- **Targets**: Main app + Share Extension
- **Capabilities**: Push Notifications, Safari Web Auth, App Groups, Siri Shortcuts

### Android

- **Min SDK**: 26 (Android 8.0)
- **Target SDK**: 35 (Android 15)
- **Build**: Gradle with Kotlin DSL

## Testing

### E2E Tests (Detox)

Located in `e2e/`:

- `onboarding.test.js` - Initial app flow tests
- `topiclist.test.js` - Topic list functionality

### Test Configurations

- iPhone 16 Pro simulator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [discourse/DiscourseMobile](https://github.com/discourse/DiscourseMobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

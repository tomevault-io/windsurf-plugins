---
trigger: always_on
description: Wrap any web app in Capacitor for iOS/Android app store deployment with RevenueCat subscriptions
---


# Capacitor App Wrapper Skill

Wraps any web app (PWA or hosted URL) in Capacitor to create native iOS and Android apps ready for app store submission, with optional RevenueCat subscription integration.

**Last Updated:** January 2026 (Capacitor 8)

## What This Skill Does

1. **Installs Capacitor** and required plugins
2. **Configures remote URL** loading for your web app
3. **Generates app icons** and splash screens from your existing assets
4. **Generates app store screenshots** automatically for iOS and Android (NEW in v2.3)
5. **Sets up RevenueCat** for in-app subscriptions (optional)
6. **Configures native platforms** (Android/iOS) with proper styling
7. **Creates Codemagic config** for iOS cloud builds
8. **Generates store listing** text ready to copy-paste
9. **Provides build commands** for both Windows and macOS

## Prerequisites

Before running this skill, ensure you have:

### Development Environment (CRITICAL - Updated for 2025/2026)

| Requirement | Version | Notes |
|-------------|---------|-------|
| **Node.js** | **22+** | Capacitor 8 requires Node 22 (NOT 18) |
| **Java JDK** | **21** | Android builds require Java 21 (NOT 17) |
| **Android Studio** | Latest | For Android SDK and emulator |
| **Xcode** | Latest | For iOS builds (macOS only) |

### Verify Your Environment

```bash
# Check Node version (must be 22+)
node --version

# Check Java version (must be 21)
java --version

# If Java is wrong, install via:
# macOS: brew install openjdk@21
# Windows: Download from adoptium.net
```

### Other Requirements
- An existing web app (either hosted URL or local PWA)
- App icons (at least 512x512 PNG)
- For iOS cloud builds: Codemagic account OR macOS with latest Xcode

## Invocation

This skill is invoked when you ask Claude to:
- "Wrap my web app in Capacitor"
- "Create a native app from my website"
- "Prepare my app for the app stores"
- "Convert my PWA to iOS/Android"

## Workflow

### Phase 1: Gather Information

Ask the user for:

1. **App URL**: The web app URL to wrap (e.g., `https://myapp.com`)
2. **App Name**: Display name (e.g., "My App")
3. **iOS Bundle ID**: Unique identifier for iOS (e.g., `app.company.appname` or `com.company.ios`)
4. **Android Package Name**: Unique identifier for Android (e.g., `com.company.appname`)
5. **Theme Colors**: Primary and accent colors (hex values)
6. **RevenueCat**: Whether to integrate subscriptions

**CRITICAL: Bundle ID Best Practices**

iOS and Android use different bundle identifier conventions and they CAN (and often SHOULD) be different:

- **iOS Bundle ID**: Apple convention often uses `app.company.appname` or `com.company.appname`
  - Examples: `app.carousel.ios`, `ai.carouselcards.app`
- **Android Package Name**: Must use reverse domain notation (e.g., `com.company.appname`)
  - Examples: `com.carouselcards.app`, `com.company.myapp`

**WARNING: Bundle IDs are PERMANENT**
- Once you publish an app to the App Store or Google Play, the bundle ID cannot be changed
- Changing it would require creating a completely new app listing
- All users, reviews, and ratings would be lost
- Choose carefully - you're committing to this forever

**Why Separate Bundle IDs?**
- Platform-specific naming conventions (iOS often uses `app.*`, Android typically `com.*`)
- Different teams or developers may manage each platform
- Allows platform-specific analytics and tracking
- Prevents conflicts when both platforms use the same identifier format

### Phase 2: Install Dependencies

```bash
# Core Capacitor (v8)
npm install @capacitor/core @capacitor/cli
npm install @capacitor/android @capacitor/ios
npm install @capacitor/splash-screen @capacitor/status-bar @capacitor/app
npm install -D @capacitor/assets

# RevenueCat (if requested) - IMPORTANT: Only use purchases-capacitor, NOT purchases-capacitor-ui
npm install @revenuecat/purchases-capacitor
# DO NOT install @revenuecat/purchases-capacitor-ui - it's NOT compatible with Capacitor 8!
```

### Phase 3: Create Configuration

Create `capacitor.config.ts`:

```typescript
import type { CapacitorConfig } from "@capacitor/cli";

const config: CapacitorConfig = {
  // NOTE: The appId in capacitor.config.ts is used as the DEFAULT for both platforms
  // We'll set the Android package name here, then override iOS bundle ID in Xcode
  appId: "{{ANDROID_PACKAGE_NAME}}",
  appName: "{{APP_NAME}}",
  webDir: "out",
  server: {
    url: "{{APP_URL}}",
    allowNavigation: ["{{DOMAIN}}", "*.{{DOMAIN}}"],
  },
  ios: {
    contentInset: "automatic",
    backgroundColor: "{{PRIMARY_COLOR}}",
  },
  android: {
    backgroundColor: "{{PRIMARY_COLOR}}",
  },
  plugins: {
    SplashScreen: {
      launchShowDuration: 2000,
      backgroundColor: "{{PRIMARY_COLOR}}",
      showSpinner: true,
      spinnerColor: "{{ACCENT_COLOR}}",
    },
  },
};

export default config;
```

**Important:** The `appId` field in capacitor.config.ts serves as the default for both platforms. However:
- Android will use this value from capacitor.config.ts (but should also be explicitly set in build.gradle)
- iOS bundle ID must be set separately in Xcode project settings (see Phase 8)

### Phase 4: Add Native Platforms

```bash
# Create placeholder out directory
mkdir -p out

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrchevyceleb/capacitor-wrap-skill](https://github.com/mrchevyceleb/capacitor-wrap-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

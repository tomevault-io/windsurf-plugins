---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rdzwx-go** is a hybrid mobile application for radiosonde (weather balloon) tracking and visualization built with Apache Cordova. The project consists of two main components:

- **rdzwx-go/**: Main Cordova application (HTML/JS frontend)
- **rdzwx-plugin/**: Custom native plugin (Kotlin for Android, Objective-C for iOS, Node.js for Electron)

## Common Development Commands

### Build Commands (from Makefile)
```bash
make run          # Build and deploy to Android device
make full         # Clean rebuild with plugin reinstall (use when plugin changes)
make plugin       # Reinstall plugin only
make el           # Build for Electron platform
make release      # Create signed release APK
```

### Core Cordova Commands
```bash
cordova build                    # Build debug version
cordova build --release         # Build release version  
cordova run android --device    # Deploy to Android device
cordova run ios --device        # Deploy to iOS device (requires Xcode on macOS)
cordova platform add android    # Add Android platform
cordova platform add ios        # Add iOS platform
cordova plugin add plugin-src/rdzwx-plugin/  # Add custom plugin
```

### Development Setup
```bash
# Initial setup after clone
cordova platform add android
cordova platform add ios        # iOS platform (requires macOS)
npm i jetifier && npx jetifier    # For AndroidX compatibility
```

### Android Emulator Commands
```bash
# List available AVDs
emulator -list-avds

# Run emulator with optimized settings (hardware GPU acceleration)
emulator -avd <avd_name> -gpu host -no-audio

# Run with snapshots for faster boot (recommended after initial setup)
emulator -avd <avd_name> -gpu host

# Note: KVM acceleration is automatically used on Linux if available
# CPU usage is high during boot (~300%) but drops to ~30% after boot completes
```

## Architecture

### Multi-Platform Hybrid Architecture
- **Frontend**: HTML5/JavaScript with Leaflet.js mapping in `rdzwx-go/www/`
- **Native Layer**: Custom Cordova plugin bridges JS to native functionality
- **Android Implementation**: Kotlin with Mapsforge offline mapping libraries
- **iOS Implementation**: Objective-C with Core Location, Network.framework, and NSNetServiceBrowser
- **Electron Implementation**: Node.js for desktop platform

### Key Technologies
- Apache Cordova v9.0.0+ for hybrid mobile framework
- Leaflet.js for interactive mapping
- Kotlin for Android native implementation
- Objective-C for iOS native implementation
- Mapsforge v0.16.0 for offline map rendering (Android)
- mDNS/Bonjour for automatic device discovery

## Project Structure

### Main Application (`rdzwx-go/`)
- `www/`: Web frontend assets (HTML, JS, CSS, images)
- `www/js/index.js`: Main application logic
- `config.xml`: Cordova configuration
- `platforms/android/`: Generated Android platform code
- `platforms/ios/`: Generated iOS platform code (Xcode project)

### Custom Plugin (`rdzwx-plugin/`)
- `src/android/`: Kotlin native implementation with AIDL interfaces
- `src/android/libs/`: JAR dependencies (Mapsforge, AndroidSVG)
- `src/ios/`: Objective-C native implementation for iOS
- `src/electron/`: Node.js/Electron implementation
- `www/rdzwx.js`: Plugin JavaScript interface
- `plugin.xml`: Plugin configuration

## Development Workflow

1. **Quick Development**: Use `make run` for frontend changes
2. **Plugin Changes**: Use `make full` when modifying native plugin code
3. **Testing**: Deploy to physical Android device or emulator
4. **Prerequisites**: Java 17, Android SDK (API 34+, build-tools 35.0.0), Node.js, Cordova CLI

### iOS Development Workflow
**IMPORTANT**: Plugin changes require reinstalling the plugin, not just building:

```bash
# After making changes to iOS plugin code in rdzwx-plugin/
cd rdzwx-go
cordova plugin remove de-dl9rdz-rdzwx && cordova plugin add plugin-src/rdzwx-plugin/
cordova build ios    # Rebuild iOS app with updated plugin
# Then test in Xcode or simulator
```

**Note**: `cordova build ios` alone does NOT copy plugin changes from `rdzwx-plugin/` to `platforms/ios/`. You must reinstall the plugin first.

### macOS Environment Setup

Required environment variables (add to `~/.zshrc`):

```bash
export JAVA_HOME=/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:/opt/homebrew/bin"
```

Install dependencies:
```bash
brew install openjdk@17 gradle nvm
brew install --cask android-studio
nvm install --lts
npm install -g cordova
```

### Android Build Requirements (cordova-android 14.x)

- **compileSdkVersion**: 35 (required by bundled AndroidX libraries)
- **targetSdkVersion**: 34 (Google Play requirement)
- **Kotlin**: 1.8.22+ (avoids duplicate class conflicts)
- **AIDL**: Enabled via `build-extras.gradle` in plugin (AGP 8.0+ disables by default)

## Release Process

1. Update version in `package.json`, `config.xml`, and `version.json`
2. Run `make release` for signed APK (requires `my-release-key.jks.gpg` keystore)
3. Primary target is Android; Electron for desktop testing

## Key Features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rdzsonde/rdzwx-go](https://github.com/rdzsonde/rdzwx-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

---
trigger: always_on
description: This file provides guidance to AI Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI Agents when working with code in this repository.

## Project Overview

The Wall is a monorepo containing a browser extension, Telegram bot, data scraper, and shared common library. The project uses pnpm workspaces with packages under `@theWallProject/*` scope.

## What This Project Does

The Wall helps users identify companies with Israeli connections through:

1. **Database of companies categorized by relationship to Israel:**
   - `h` (HeadQuarter), `f` (Founder), `i` (Investor), `u` (URL)
   - BDS categories: `BDS_PRIO` (Consumer boycott priority), `BDS_GRASS` (Grassroots organic), `BDS_PRESSURE` (Pressure targets)

2. **Browser Extension**: Monitors websites, social media profiles (LinkedIn, Facebook, Twitter/X, Instagram, GitHub, YouTube, TikTok, Threads), and job listings. Shows visual warnings and suggests alternatives.

3. **Telegram Bot**: URL/company checking via inline queries

4. **Data Pipeline**: Aggregates from Crunchbase, BuyIsraeliTech, and manual sources

The extension uses different strategies: full-page overlays for direct visits, inline filtering for feeds (e.g., LinkedIn jobs), dismissible notifications.

## Essential Commands

```bash
# Build (must follow dependency order: common → scrapper → addon → telegram-bot)
pnpm build                      # All packages
pnpm build:common              # Must build first
pnpm build:chrome              # Chrome-specific addon

# Development
pnpm dev                       # Addon dev mode

# Testing
pnpm test:addon                # All addon tests (requires built extension)
pnpm test:addon:e2e            # E2E tests only

# Data Pipeline
pnpm data                      # Unified data menu (validate, add, delete, quick-verify, AI extract, apply, full pipeline)
# For non-interactive regeneration (useful after editing MANUAL.json or manualAdditions.ts):
cd packages/scrapper && pnpm exec ts-node src/tasks/regenerate_db.ts

# Linting & Commits
pnpm lint                      # Check all
pnpm lint:fix                  # Auto-fix
pnpm commit                    # VibElint commit workflow (required by hooks)

# Telegram Bot
pnpm bot:dev                   # Development (polling)
pnpm bot:deploy                # Deploy to production

# Android App
cd packages/android
pnpm build                     # Debug APK
pnpm build:release             # Release APK (unsigned)
pnpm release                   # Full release workflow (signed APK)
pnpm release:patch             # Bump patch version + release
pnpm release:minor             # Bump minor version + release
pnpm release:major             # Bump major version + release
pnpm clean                     # Clean build artifacts
pnpm lint                      # Run Android lint (MUST pass with zero errors)
pnpm test                      # Run all tests + record screenshots + lint (no emulator needed)
pnpm test:update               # Update screenshot baselines
pnpm test:verify               # Verify screenshots match baselines (CI mode)
pnpm test:compare              # Generate HTML diff report for screenshot changes
pnpm test:screenshots          # Generate fastlane Play Store screenshots for ALL supported languages
pnpm install:prod              # Install release APK to connected device via USB
pnpm validate:metadata         # Validate Play Store metadata
pnpm screenshots               # Alias for test:screenshots
pnpm adb:trigger_scan          # Force trigger background scan
pnpm adb:logcat                # View app logs
```

### Android Build Requirements

The Android app requires:

- **Bash**: Git Bash on Windows, or native bash on macOS/Linux
- **Java 17+**: Auto-detected from Android Studio's bundled JDK, or set `JAVA_HOME`
- **Gradle 9.1.0**: Configured in `gradle/wrapper/gradle-wrapper.properties`
- **Android SDK**: Set `ANDROID_HOME` or configure in `local.properties`

The build scripts auto-detect Android Studio's JDK on Windows (`C:\Program Files\Android\Android Studio\jbr`).

**IMPORTANT**: Always use `pnpm build` from `packages/android` directory instead of running gradlew directly. The pnpm scripts handle environment setup (JAVA_HOME, paths) automatically.

### Android Release Workflow

The Android app uses a streamlined release workflow:

1. **Version management**: `version.properties` tracks VERSION_CODE and VERSION_NAME
2. **Signing**: Configure `keystore.properties` (see `keystore.properties.template`)
3. **Screenshots**: Script prompts during release; run separately with `pnpm test:screenshots` (JVM-based, no emulator needed)
4. **Release**: Run `pnpm release:patch` (or minor/major) to bump version and build signed APK.
5. **Output**: Signed APK is placed in `release-output/thewall-v{version}.apk`

**Screenshot generation (JVM-based, all languages):**

- `pnpm test:screenshots` generates 4 phone + 4 tablet screenshots per language
- Auto-discovers languages from `SupportedLanguage.entries` — adding a new language to the enum automatically generates screenshots for it
- Output: `fastlane/metadata/android/{locale}/images/{phone|tablet}Screenshots/{1-4}.png`
- Uses Robolectric + Roborazzi (no emulator needed)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theWallProject/mono](https://github.com/theWallProject/mono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

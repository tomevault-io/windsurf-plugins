---
trigger: always_on
description: This is an open-source Android application for privacy-friendly location tracking, sharing, and device management. Built with Kotlin, Jetpack Compose, and modern Android APIs (minSdk 24, targetSdk 35 / Android 15). Developed with Android Studio.
---

# Cursor Rules for miataru Android App

## Project Context
This is an open-source Android application for privacy-friendly location tracking, sharing, and device management. Built with Kotlin, Jetpack Compose, and modern Android APIs (minSdk 24, targetSdk 35 / Android 15). Developed with Android Studio.

## Code Style & Standards
- Use Kotlin and Jetpack Compose best practices
- Follow Material 3 and Android UX guidelines
- Maintain a privacy-first approach
- Use English for all code comments and strings
- Implement robust error handling and battery optimization
- Prefer coroutines and flows; be lifecycle-aware
- Enforce null-safety and avoid unchecked casts

## Architecture Guidelines
- Use MVVM with `ViewModel` and `StateFlow`
- Keep composables lightweight; delegate business logic to managers/repositories
- Use DataStore for settings persistence
- Use Room (or serialized files) for device/entities persistence
- Handle runtime permissions gracefully (foreground and background location)
- Use Hilt for dependency injection

## Key Components
- `LocationTrackingManager` and `LocationTrackingService` (FusedLocationProvider + Foreground Service)
- `SettingsRepository` (DataStore)
- `KnownDeviceStore` and optional `DeviceGroupStore` (Room/DataStore)
- `MiataruApiClient` (Retrofit or Ktor + Kotlinx Serialization)
- QR scanning/generation (ML Kit or ZXing)
- Map UI (Google Maps Compose)
- Background tasks (WorkManager)

## Development Workflow
- Test on phones and tablets (emulator and physical devices)
- Consider battery optimization and background execution limits
- Maintain privacy compliance and Google Play policies
- Use Android Studio with Gradle builds

## Common Patterns
- Use `collectAsStateWithLifecycle` to observe flows in composables
- Implement a clear onboarding flow with runtime permission requests
- Handle background/foreground transitions and process death
- Manage device identifiers securely in app-private storage

## Localization Guidelines

### String Localization
ALWAYS use Android string resources with proper English descriptions for translators. Reference strings via `stringResource` in Compose or `getString` in Views.

#### ✅ Correct (Compose)
```kotlin
Text(stringResource(id = R.string.no_groups_available_create_new))
```

#### ❌ Incorrect
```kotlin
Text("No groups available. Create a new group to get started.")
```

### Localization Key Naming Convention
- Use descriptive, lowercase keys with underscores
- Follow the pattern: `context_action_description`
- Examples:
  - `group_empty_add_devices_message`
  - `no_groups_available_create_new`
  - `edit_device`
  - `delete_group`

### Comment Guidelines
- In `strings.xml`, place an XML comment above the string to guide translators
- Use proper English grammar and punctuation
- Make comments self-explanatory; avoid technical jargon

### Adding New Localized Strings
1. Use `stringResource(R.string.key)` in Kotlin code
2. Add `<string name="key">Value</string>` to `app/src/main/res/values/strings.xml`
3. Provide translations in `res/values-de/strings.xml` and `res/values-ja/strings.xml`

## File Naming Conventions

### Module and Directory Structure
```
app/
└── src/main/
    ├── java/com/miataru/
    │   ├── ui/                 # Compose UI (screens, components)
    │   ├── location/           # Location managers & foreground service
    │   ├── settings/           # Settings repository & models
    │   ├── data/               # Room/DataStore, repositories, entities
    │   ├── network/            # Miataru API client & DTOs
    │   ├── di/                 # Hilt modules
    │   └── util/               # Utilities
    └── res/                    # Resources & localization
```

### File Naming
- Composables: `FeatureScreen.kt`, `FeatureDialog.kt`, `FeatureCard.kt`
- ViewModels: `FeatureViewModel.kt`
- Repositories: `FeatureRepository.kt`
- Services/Managers: `LocationTrackingService.kt`, `LocationTrackingManager.kt`
- Data models/entities: `Device.kt`, `DeviceEntity.kt`, `LocationSample.kt`

## Copyright Header Requirements

### Mandatory Copyright Header
ALWAYS add the following header at the very top of every new Kotlin file, before the `package` declaration:

```kotlin
/*
 * Copyright (c) 2013-2025, Daniel Kirstenpfad, www.miataru.com
 *
 * [FileName].kt
 * miataru
 *
 * Created by Daniel Kirstenpfad on [CurrentDate].
 */
```

#### ✅ Correct Implementation
```kotlin
/*
 * Copyright (c) 2013-2025, Daniel Kirstenpfad, www.miataru.com
 *
 * MyNewScreen.kt
 * miataru
 *
 * Created by Daniel Kirstenpfad on 2025-01-25.
 */

package com.miataru.ui
// ... rest of the file content
```

#### ❌ Incorrect Implementation
```kotlin
package com.miataru.ui
// Missing copyright header
// ... rest of the file content
```

### Header Guidelines
- Place the header at the very top, before any `package` or `import`
- Replace `[FileName]` with the actual filename (e.g., `MyNewScreen.kt`)
- Replace `[CurrentDate]` with YYYY-MM-DD (e.g., 2025-01-25)
- Keep the exact copyright text and format

## Architecture Patterns

### MVVM with Jetpack

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miataru) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->

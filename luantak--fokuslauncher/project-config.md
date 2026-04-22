---
trigger: always_on
description: Fokus Launcher is an Android launcher application designed for a minimal, clean user experience. The app is built with modern Android development practices using Kotlin, Jetpack Compose, and Material 3.
---

# Fokus Launcher - Copilot Instructions

## Project Overview

Fokus Launcher is an Android launcher application designed for a minimal, clean user experience. The app is built with modern Android development practices using Kotlin, Jetpack Compose, and Material 3.

**Tech Stack:**
- Language: Kotlin
- UI Framework: Jetpack Compose with Material 3
- Dependency Injection: Hilt
- Database: Room
- Preferences: DataStore
- Navigation: Navigation Compose
- Minimum SDK: 26 (Android 8.0)
- Compile/Target SDK: 36
- Build Tool: Gradle 9.1.0 with AGP 9.0.0
- Required JDK: 21 (Temurin)

## Build & Test Instructions

### Environment Setup

**CRITICAL:** Always use JDK 21. The project requires JDK 21 as specified in the GitHub Actions workflow. The `gradle/gradle-daemon-jvm.properties` file specifies a JetBrains JDK vendor, but in environments where this is not available (like CI with Temurin), the build may require toolchain auto-provisioning or manual JDK configuration.

Android Studio is optional. The Gradle commands below are sufficient for builds
and unit tests.

### Building the Project

```bash
# Build debug APK
./gradlew :app:assembleDebug

# Build release APK (requires signing config)
./gradlew :app:assembleRelease

# Install debug APK on connected device
./gradlew installDebug
```

**Build Outputs:**
- Debug APK: `app/build/outputs/apk/debug/`
- Release APK: `app/build/outputs/apk/release/`

**Build Notes:**
- Clean builds are recommended when switching between branches or after major dependency changes
- The first build downloads Gradle wrapper and dependencies, which can take several minutes
- Release builds require signing configuration via environment variables or `gradle.properties`:
  - `ANDROID_KEYSTORE_PATH`
  - `ANDROID_KEYSTORE_PASSWORD`
  - `ANDROID_KEY_ALIAS`
  - `ANDROID_KEY_PASSWORD`

### Running Tests

```bash
# Run unit tests
./gradlew testDebugUnitTest

# Run instrumented tests (requires connected device/emulator)
./gradlew connectedAndroidTest

# Run specific test class
./gradlew test --tests "com.lu4p.fokuslauncher.ui.home.HomeViewModelTest"
```

**Test Locations:**
- Unit tests: `app/src/test/java/`
- Instrumented tests: `app/src/androidTest/java/`

**Testing Framework:**
- JUnit 4 for test structure
- MockK for mocking
- Turbine for Flow testing
- Coroutines Test for suspend function testing
- Hilt for dependency injection in tests

### Linting & Code Quality

```bash
# Run lint checks
./gradlew lint

# Format code (if ktlint is configured)
./gradlew ktlintFormat
```

**Note:** The project uses Kotlin official code style as specified in `gradle.properties`.

## Project Structure

### Source Code Layout

```
app/src/main/java/com/lu4p/fokuslauncher/
├── FokusLauncherApp.kt          # Application class with Hilt setup
├── MainActivity.kt               # Single activity with Compose setup
├── data/                        # Data layer
│   ├── database/                # Room database entities and DAOs
│   ├── local/                   # Local data sources (PreferencesManager)
│   ├── model/                   # Data models
│   └── repository/              # Repository implementations
├── di/                          # Hilt dependency injection modules
├── ui/                          # UI layer (Compose screens & ViewModels)
│   ├── components/              # Reusable Compose components
│   ├── drawer/                  # App drawer screen
│   ├── home/                    # Home screen
│   ├── navigation/              # Navigation graph
│   ├── onboarding/              # First-run onboarding
│   ├── settings/                # Settings screen
│   └── theme/                   # Material 3 theme configuration
└── utils/                       # Utility classes
```

### Configuration Files

- `build.gradle.kts` (root): Top-level build configuration
- `app/build.gradle.kts`: App module build configuration with dependency definitions
- `gradle/libs.versions.toml`: Centralized dependency version catalog
- `gradle.properties`: Gradle and Android build properties
- `app/proguard-rules.pro`: ProGuard rules for release builds
- `app/src/main/AndroidManifest.xml`: App manifest with permissions and launcher configuration

### Key Architectural Patterns

1. **MVVM Architecture**: UI screens use ViewModels to manage state and business logic
2. **Repository Pattern**: Data layer uses repositories to abstract data sources
3. **Dependency Injection**: Hilt provides dependencies throughout the app
4. **Unidirectional Data Flow**: ViewModels expose StateFlows that UI observes
5. **Compose UI**: Declarative UI with Material 3 components

## Continuous Integration

The project uses GitHub Actions for CI/CD (`.github/workflows/android-build-and-release.yml`):

**Build Workflow (on push/PR):**
1. Checkout code
2. Set up JDK 21 (Temurin)
3. Set up Gradle with caching
4. Set up Android SDK (platform-tools, platforms;android-36, build-tools;36.0.0)
5. Build debug APK with `./gradlew :app:assembleDebug`
6. Upload APK as artifact

**Release Workflow (on GitHub release):**
1. Same setup as build workflow
2. Validate signing secrets are present
3. Decode keystore from base64

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luantak/FokusLauncher](https://github.com/luantak/FokusLauncher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

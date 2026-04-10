---
trigger: always_on
description: Project development guidelines
---

Project development guidelines

## Overview

Kotlin Multiplatform (KMP) workspace with 3 modules:

- **composeApp**: Compose Multiplatform UI (Android, Desktop, iOS)
- **shared**: KMP shared library
- **server**: JVM Ktor server

**Stack**: Gradle, Kotlin, Compose Multiplatform, AGP, Ktor, SqlDelight, Valkyrie

## Build Commands

**Environment Notes**:

- **MSYS/Git Bash (native Windows)**: Use `./gradlew.bat` directly
- **WSL**: Use `cmd.exe /c gradlew.bat` prefix (Windows paths in output)

### Core Tasks

- **All tests**: `gradlew test`
- **Build all**: `gradlew build`
- **Run desktop**: `gradlew :composeApp:run`
- **Run server (dev)**: `gradlew -Pdevelopment :server:run`

### Module-Specific Tasks

- **Shared tests**: `gradlew :shared:allTests`
- **ComposeApp tests**: `gradlew :composeApp:allTests`
- **Android instrumented tests**: `gradlew :composeApp:connectedAndroidTest`
- **Server tests**: `gradlew :server:test`
- **Android APK**: `gradlew :composeApp:assembleDebug`
- **Desktop distribution**: `gradlew :composeApp:packageDistributionForCurrentOS`

## Test Structure

- **shared**: Core logic lives in `shared/src/commonMain`; use the composeApp tests to cover DB behaviour.
- **composeApp**: Shared tests live in `composeApp/src/commonTest`. The expect/actual `BaseTest` + `TestContext` wiring
  lets the same tests run on every target:
    - Android JVM (`gradlew :composeApp:androidUnitTest`) uses Robolectric to provide an Android context.
    - Android instrumented (`gradlew :composeApp:connectedAndroidTest`) reuses the same `commonTest` sources; requires
      an emulator/device and network access for DB download tests.
    - Desktop JVM (`gradlew :composeApp:desktopTest`) exercises the same tests against the JDBC driver.
    - iOS simulator (`gradlew :composeApp:iosSimulatorArm64Test`) runs the tests on macOS; native targets are skipped
      elsewhere.
- **server**: `server/src/test` (ktor-server-test-host, kotlin-test-junit)

### Adding Tests

- Default to `composeApp/src/commonTest` so logic executes on Android JVM, Android instrumented, Desktop, and iOS.
- Extend `BaseTest` for KMP tests; it injects the platform context through `TestContext`.
- If a test needs target-specific setup, update the relevant `TestContext.<platform>.kt` actual or add a new
  expect/actual helper alongside `BaseTest`.
- Only add platform-specific test source sets when behaviour truly diverges (e.g., Android UI instrumentation);
  otherwise keep coverage centralized in `commonTest`.

## Configuration

- **Versions**: All in `gradle/libs.versions.toml`
- **JVM target**: 11 for Android/Shared
- **iOS**: Disabled on non-macOS (expected behavior)
- **Android SDK**: compileSdk=36, minSdk=24, targetSdk=36
- **Java version**: Use Java 21 (via sdkman: `sdk use java 21.0.9-amzn`). Java 25+ may cause Kotlin compatibility
  issues.
- **Data version**: `DataDbManager.VERSION = "v11"`; when bumping, upload DBs under the new prefix in the GCS bucket and
  keep the version in sync.

## Code Structure

- **KMP source sets**: commonMain/commonTest for shared code
- **shared module**: Cross-platform APIs (e.g., Greeting.greet(), Constants.kt)
- **Compose UI**: composeApp/src/commonMain
- **Android namespace**: com.slovy.slovymovyapp
- **Server main**: com.slovy.slovymovyapp.ApplicationKt

### Compose UI workflow

- Split each screen into a thin stateful entry point and a stateless composable that renders a `UiState` data model;
  previews/tests should target the stateless layer.
- Keep all mutable UI flags (loading, expanded sections, dialog visibility, etc.) inside the `UiState`; avoid
  `remember`/`rememberSaveable` inside rendering composables.
- Provide explicit callbacks (`onToggle`, `onRetry`, …) so the orchestrator can mutate the `UiState` while previews pass
  no-op lambdas.
- Add preview functions for every meaningful `UiState` variant (content, loading, error, empty) so designers/devs can
  inspect layouts without runtime wiring.
- When deriving default UI state from domain models (e.g., `LanguageCard`), add helper mappers (`toUiState()`) rather
  than embedding logic inside composables.

#### Preview Functions

- All `@Preview` functions must support both light and dark themes using the themed preview pattern.
- Use `@PreviewParameter(ThemePreviewProvider::class) isDark: Boolean` to receive theme parameter.
- Wrap preview content with `ThemedPreview(darkTheme = isDark) { ... }` to apply the theme.
- Import required types: `PreviewParameter` from `androidx.compose.ui.tooling.preview.PreviewParameter`.
- The `ThemePreviewProvider` and `ThemedPreview` are defined in
  `composeApp/src/commonMain/kotlin/com/slovy/slovymovyapp/ui/Preview.kt`.

Example:

```kotlin
@Preview
@Composable
private fun MyScreenPreview(
    @PreviewParameter(ThemePreviewProvider::class) isDark: Boolean
) {
    ThemedPreview(darkTheme = isDark) {
        MyScreenContent(state = MyUiState(...))
    }
}
```

### ViewModel pattern

- Every screen should use a ViewModel to manage state and survive configuration changes.
- Create a `<ScreenName>ViewModel` class that extends `ViewModel` and holds the screen's `UiState`.
- State should be exposed as `var state by mutableStateOf(...)` with `private set`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slovymovy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

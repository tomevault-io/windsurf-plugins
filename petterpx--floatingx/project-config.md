---
trigger: always_on
description: FloatingX is an Android library that provides flexible and powerful floating window solutions, supporting system-level, app-level, and local floating windows with JetPack Compose support.
---

# FloatingX - Android Floating Window Library

FloatingX is an Android library that provides flexible and powerful floating window solutions, supporting system-level, app-level, and local floating windows with JetPack Compose support.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Prerequisites and Setup
- Install Java 17 (OpenJDK recommended): Required for Android Gradle Plugin 8+
- Ensure Android SDK is available (minimum SDK 21, target/compile SDK 34)  
- **CRITICAL**: Ensure internet access to Google Maven and Gradle Plugin repositories
- **CRITICAL**: Set JVM heap size: `export GRADLE_OPTS="-Xmx4g -XX:MaxMetaspaceSize=1g"`

### Bootstrap, Build, and Test Repository
**NEVER CANCEL builds or tests - they may take significant time. Use appropriate timeouts.**

```bash
# Grant execute permission (required on fresh clone)
chmod +x gradlew

# Clean and build all modules - NEVER CANCEL: Takes 3-8 minutes. Set timeout to 15+ minutes.
./gradlew clean build publishToMavenLocal -PisPublish=false -PversionName=1.0.0

# Run tests - NEVER CANCEL: Takes 2-5 minutes. Set timeout to 10+ minutes.
./gradlew test

# Run instrumented tests (if device/emulator available) - NEVER CANCEL: Takes 5-15 minutes. Set timeout to 20+ minutes.
./gradlew connectedAndroidTest
```

### Build the Demo Application
```bash
# Build debug APK - NEVER CANCEL: Takes 2-4 minutes. Set timeout to 10+ minutes.
./gradlew app:assembleDebug

# Install and run on connected device/emulator
./gradlew app:installDebug
adb shell am start -n com.petterp.floatingx.app/.MainActivity
```

### Library Publication (Development)
```bash
# Publish to local Maven repository - NEVER CANCEL: Takes 3-6 minutes. Set timeout to 15+ minutes.
./gradlew publishToMavenLocal -PisPublish=false -PversionName=1.0.0-SNAPSHOT

# For release publication (requires signing keys)
./gradlew publishAllPublicationsToMavenCentralRepository -PisPublish=true -PversionName=X.Y.Z
```

## Validation Requirements

### Manual Testing Scenarios
Always perform these validation steps after making changes to core library functionality:

1. **System Floating Window Test**:
   - Build and install demo app: `./gradlew app:installDebug`
   - Launch app and navigate to "System Floating Window" option
   - Grant SYSTEM_ALERT_WINDOW permission when prompted  
   - Use `TestActivity` → "进入多浮窗页面(测试多浮窗功能)" for comprehensive testing
   - Verify floating window appears and is draggable across the screen
   - Test edge absorption, boundary bounce, and multi-touch

2. **App-Level Floating Window Test**:
   - Navigate to app-level floating window demo via `MainActivity`
   - Test local floating windows: "显示局部悬浮窗" button
   - Verify floating window works within app boundaries
   - Test rotation, app switching, and lifecycle scenarios

3. **Local/Scoped Floating Window Test**:
   - Use `TestActivity` → "进入局部悬浮窗页面-(测试api功能)" (`ScopeActivity`)
   - Test Activity, Fragment, and ViewGroup scoped windows
   - Verify floating windows appear only within their designated containers
   - Test view lifecycle and cleanup

4. **Compose Integration Test**:
   - Test Compose floating windows functionality in demo app
   - Verify `enableComposeSupport()` call is working (see `FxComposeSimple.kt`)
   - Test Compose UI rendering within floating windows
   - Check system floating windows with Compose content

5. **Edge Cases and Special Scenarios**:
   - Test immersive mode: "进入无状态栏页面-(测试状态栏影响)" (`ImmersedActivity`)
   - Test RecyclerView interaction: "进入recyclerView测试页面" (`SimpleRvActivity`)
   - Test blacklist functionality: "进入黑名单页面(该页面禁止展示浮窗1)" (`BlackActivity`)

**Expected Outcomes**:
- All floating windows should be draggable and responsive
- No crashes during permission requests or lifecycle changes
- Proper cleanup when Activities/Fragments are destroyed
- Floating windows should respect their scope boundaries

### Build Validation
Always run before committing changes:
```bash
# Lint check - NEVER CANCEL: Takes 1-3 minutes. Set timeout to 5+ minutes.
./gradlew lint

# Code style check with Detekt (configured in check/detekt/detekt.yml)
./gradlew detekt

# Full CI validation - NEVER CANCEL: Takes 5-10 minutes. Set timeout to 20+ minutes.
./gradlew clean build publishToMavenLocal -PisPublish=false -PversionName=1.0
```

## Project Structure and Navigation

### Key Modules
- **`app/`** - Demo application showcasing all FloatingX features
  - Entry point: `com.petterp.floatingx.app.MainActivity`
  - Test activities: `com.petterp.floatingx.app.TestActivity`
  - Example implementations in `app/src/main/java/com/petterp/floatingx/app/kotlin/`

- **`floatingx/`** - Core floating window library
  - Main API: `com.petterp.floatingx.FloatingX`
  - Core classes: `src/main/java/com/petterp/floatingx/`
  - **Important**: Always check this module when modifying core functionality

- **`floatingx_compose/`** - JetPack Compose support extension
  - Compose integration: `src/main/java/com/petterp/floatingx/compose/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Petterpx/FloatingX](https://github.com/Petterpx/FloatingX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

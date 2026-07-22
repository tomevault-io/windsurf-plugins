---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KiteUI is a Kotlin Multiplatform UI framework inspired by Solid.js that uses native view components on each platform (Android, iOS, JVM, JS/Web). It emphasizes small binary sizes, fine-grained reactivity, semantic theming, and URL-based navigation for web compatibility.

**Key Design Principles:**
- Uses fine-grained reactivity based on Solid.js (not Compose-style recomposition)
- Navigation is URL-based for web compatibility
- Semantic theming system (not direct styling)
- Platform-native view components (not canvas rendering)
- Custom lightweight network client (not Ktor for production)

## Project Structure

This is a multi-module Gradle project:

- **library/** - Core KiteUI framework code
  - `src/commonMain/` - Platform-independent code
  - `src/androidMain/`, `src/iosMain/`, `src/jsMain/`, `src/jvmMain/` - Platform-specific implementations
  - Key packages:
    - `views/` - View components and modifiers
    - `navigation/` - Routing and page system
    - `reactive/` - Reactive state management
    - `models/` - Theme, styling, and data models
- **example-app/** - Demo application showcasing KiteUI features
- **gradle-plugin/** - Gradle plugin for KiteUI projects
- **buildSrc/** - Build configuration utilities

## Development Commands

### Building
```bash
# Build all modules
./gradlew build

# Build specific module
./gradlew :library:build
./gradlew :example-app:build

# Publish to Maven Local
./gradlew publishToMavenLocal
```

### Testing

See **[docs/RUNNING_TESTS.md](docs/RUNNING_TESTS.md)** for the full guide including prerequisites, gotchas, and how to write tests.

```bash
# Fastest — no external tools needed
./gradlew :example-app:jvmSsrTest

# Android (Robolectric, no device needed)
./gradlew :example-app:testDebugUnitTest

# iOS (requires running Simulator)
./gradlew :example-app:iosSimulatorArm64Test

# JS/Web (requires Chrome)
./gradlew :example-app:jsBrowserTest

# All platforms at once
./gradlew :example-app:allTests

# Filter to a specific test class (JVM SSR and Android only)
./gradlew :example-app:jvmSsrTest --tests "*.MyTestClass"
./gradlew :example-app:testDebugUnitTest --tests "*.MyTestClass"
```

Substitute `:library:` for `:example-app:` to run library tests instead.

### Running Example App
```bash
# Run JS/Web version with Vite (development)
# IMPORTANT: Always use Gradle tasks to run the dev server, NOT manual HTTP servers
./gradlew :example-app:viteRun

# Run JS/Web version (production build)
# Use run configuration: "ExampleJSRun prod"

# Run Android version
# Use Android run configuration: "example-app"

# Run JVM version
./gradlew :example-app:jvmRun
```

**Note for Claude:** When testing JS/Web changes, always use `./gradlew :example-app:viteRun` to start the dev server. Do NOT use Python HTTP servers or other manual servers - they don't handle SPA routing correctly.

### Publishing
```bash
# Publish to LightningKite repository
./gradlew publishAllPublicationsToLightningKiteRepository
```

## Architecture Patterns

### Pages and Navigation

Pages implement the `Page` interface and are annotated with `@Routable`:

```kotlin
@Routable("your/path")
object YourPage : Page {
    override fun ViewWriter.render(): Unit = run {
        // UI code
    }
}
```

For pages with parameters:
```kotlin
@Routable("items/{id}")
class ItemDetailPage(val id: String) : Page {
    override fun ViewWriter.render(): Unit = run {
        // Access id parameter
    }
}
```

Navigate using `pageNavigator.navigate(SomePage)` or use `link` components.

### ViewWriter and Component Creation

UI is built using `ViewWriter` extension functions. Components return `Unit` for modifier chaining:

```kotlin
fun ViewWriter.myComponent(): Unit = col {
    text("Hello")
    button {
        text("Click me")
    }
}
```

Common containers: `row`, `col`, `frame`, `rowCollapsingToColumn`

### Modifiers

Modifiers are applied with the `-` operator. Order matters: Position > Visibility > Scroll > Theme

```kotlin
centered - scrolling - card - col {
    // content
}
```

Common modifiers:
- Layout: `centered`, `expanding`, `weight(f)`, `sizeConstraints(...)`
- Spacing: `padded`
- Scrolling: `scrolling`
- Theme: `card`, `important`, `fieldTheme`, etc.

### Reactivity System

KiteUI uses fine-grained reactivity, not recomposition:

**Property** - Basic reactive container:
```kotlin
val email = Property("")
textInput { content bind email }
```

**Reactive functions** - Auto-update when dependencies change:
```kotlin
text {
    ::content { "Email: ${email()}" }
}
```

**shared** - Cached computed value:
```kotlin
val fullName = shared { "${firstName()} ${lastName()}" }
```

**LazyProperty** - Computed value that can be overridden:
```kotlin
val calculated = LazyProperty { base() * multiplier() }
calculated.value = 100.0  // Override
calculated.reset()        // Back to calculation
```

**LateInitProperty** - For values not available at declaration:
```kotlin
val userData = LateInitProperty<UserData>()
// Components show loading until value is set
userData.value = fetchedData
```

### Theming


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lightningkite/kiteui](https://github.com/lightningkite/kiteui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

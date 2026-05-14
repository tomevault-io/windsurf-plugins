---
trigger: always_on
description: **titan2keyboard** is a modern Input Method Editor (IME) keyboard specifically designed for the Unihertz Titan 2 device. This is an Android keyboard application that provides an enhanced typing experience optimized for the physical QWERTY keyboard found on the Titan 2.
---

# CLAUDE.md - AI Assistant Guide for titan2keyboard

## Project Overview

**titan2keyboard** is a modern Input Method Editor (IME) keyboard specifically designed for the Unihertz Titan 2 device. This is an Android keyboard application that provides an enhanced typing experience optimized for the physical QWERTY keyboard found on the Titan 2.

## Project Status

This is a newly initialized project. The codebase is in early development with minimal structure established.

## Design Philosophy

**Clean Sheet, Modern Implementation**

This project is built from the ground up using the latest Android technologies and best practices:

- **No Legacy Code**: 100% modern implementation, no backwards compatibility concerns
- **Android 15 First**: Targets latest Android platform (API 35), minimum Android 14 (API 34)
- **Kotlin-Only**: Pure Kotlin codebase, no Java
- **Modern Toolchain**: Latest Gradle, AGP, KSP (no KAPT), version catalogs
- **Jetpack Compose**: Declarative UI with Material Design 3
- **Clean Architecture**: Proper separation of concerns (UI → Domain → Data)
- **Reactive Patterns**: Coroutines, Flow, StateFlow throughout
- **Dependency Injection**: Hilt for compile-time DI
- **Type Safety**: Leverage Kotlin's type system fully
- **Performance**: Built for Android 15's performance features and R8 optimization
- **Developer Experience**: Fast builds, clear architecture, maintainable code

This approach allows us to use the best tools and patterns without being constrained by legacy requirements.

## Technology Stack

### Core Technologies
- **Platform**: Android 15 (API 35)
- **Language**: Kotlin (100% Kotlin, no Java)
- **Build System**: Gradle 8.x with Android Gradle Plugin 8.x (Kotlin DSL)
- **Target Device**: Unihertz Titan 2 (physical QWERTY keyboard)
- **Minimum SDK**: 34 (Android 14) - Clean sheet, no backwards compatibility needed
- **Target SDK**: 35 (Android 15)
- **Compile SDK**: 35

### Modern Android Architecture & Libraries

**UI Framework**
- **Jetpack Compose** - Modern declarative UI (Material Design 3)
- Compose UI for settings and configuration screens
- Material Design 3 components and theming

**Architecture Components**
- **MVVM/MVI Architecture** - Modern unidirectional data flow
- **ViewModel** - UI state management with lifecycle awareness
- **StateFlow/SharedFlow** - Reactive state management (prefer over LiveData)
- **Lifecycle** - Lifecycle-aware components

**Dependency Injection**
- **Hilt** - Modern dependency injection framework
- Compile-time DI for better performance

**Asynchronous Programming**
- **Kotlin Coroutines** - All async operations
- **Flow** - Reactive streams for data
- **Dispatchers** - Proper thread management (Main, IO, Default)

**Data Persistence**
- **DataStore** - Modern preferences storage (replaces SharedPreferences)
- **Room** (if needed) - Type-safe database access with Kotlin coroutines support

**Build & Dependencies**
- **Version Catalog** (libs.versions.toml) - Centralized dependency management
- **Kotlin Symbol Processing (KSP)** - Replace kapt for faster builds

### Key Android Components
- `InputMethodService` - Core IME service
- Android Input Method Framework (latest APIs)
- Hardware keyboard event handling
- Modern accessibility APIs

## Expected Project Structure

```
titan2keyboard/
├── app/                          # Main application module
│   ├── src/
│   │   ├── main/
│   │   │   ├── kotlin/com/titan2keyboard/
│   │   │   │   ├── Titan2KeyboardApp.kt  # Application class with Hilt
│   │   │   │   ├── di/          # Dependency injection modules
│   │   │   │   │   ├── AppModule.kt
│   │   │   │   │   └── DataModule.kt
│   │   │   │   ├── ime/         # IME service implementation
│   │   │   │   │   ├── Titan2InputMethodService.kt
│   │   │   │   │   └── KeyEventHandler.kt
│   │   │   │   ├── domain/      # Business logic layer
│   │   │   │   │   ├── model/   # Domain models
│   │   │   │   │   ├── repository/  # Repository interfaces
│   │   │   │   │   └── usecase/ # Use cases
│   │   │   │   ├── data/        # Data layer
│   │   │   │   │   ├── repository/  # Repository implementations
│   │   │   │   │   ├── datastore/   # DataStore preferences
│   │   │   │   │   └── model/   # Data models/DTOs
│   │   │   │   ├── ui/          # Presentation layer (Compose)
│   │   │   │   │   ├── theme/   # Material3 theme
│   │   │   │   │   ├── settings/    # Settings screens
│   │   │   │   │   │   ├── SettingsScreen.kt
│   │   │   │   │   │   └── SettingsViewModel.kt
│   │   │   │   │   └── components/  # Reusable Compose components
│   │   │   │   └── util/        # Utility classes and extensions
│   │   │   ├── res/
│   │   │   │   ├── values/      # Strings, colors, dimensions (Compose-based)
│   │   │   │   ├── xml/         # IME method definitions, input method
│   │   │   │   └── drawable/    # Vector drawables, icons
│   │   │   └── AndroidManifest.xml
│   │   ├── test/                # Unit tests (JUnit 5, Mockk)
│   │   └── androidTest/         # Instrumentation tests (Compose UI tests)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Divefire/titan2keyboard](https://github.com/Divefire/titan2keyboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

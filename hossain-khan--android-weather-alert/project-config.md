---
trigger: always_on
description: This document provides context and coding guidelines for the Weather Alert Android App to help GitHub Copilot generate appropriate code suggestions.
---

# GitHub Copilot Custom Instructions

This document provides context and coding guidelines for the Weather Alert Android App to help GitHub Copilot generate appropriate code suggestions.

## Project Overview

Weather Alert is a modern Android application that provides focused weather notifications based on user-configured thresholds. The app helps users prepare for specific weather conditions like snow and rain by sending timely alerts when conditions meet their criteria.

### Key Features
- Custom alerts for snowfall and rainfall with user-defined thresholds
- Multiple weather data sources (OpenWeatherMap, Tomorrow.io, OpenMeteo, WeatherAPI)
- Configurable alert frequency (6, 12, or 18 hours)
- Rich notifications with actionable information
- Minimalist tile-based UI design
- Background processing with WorkManager

## Architecture & Tech Stack

### Core Architecture
- **UI Framework**: Jetpack Compose with Material 3 design system
- **Architecture Pattern**: Circuit UDF (Unidirectional Data Flow) by Slack
- **Dependency Injection**: Metro (modern Kotlin-first DI framework)
- **Navigation**: Jetpack Navigation Compose
- **State Management**: Circuit Presenters and UI components

### Data Layer
- **Database**: Room with KSP code generation
- **Preferences**: DataStore (both Core and Preferences)
- **Network**: Retrofit 3 + OkHttp 4 with logging interceptor
- **JSON Parsing**: Moshi with Kotlin codegen
- **Serialization**: Kotlin Serialization + Moshi
- **Error Handling**: EitherNet for type-safe network results

### Background Processing
- **WorkManager**: For periodic weather checks and alert generation
- **Notifications**: Android notification system with rich content support

### Testing & Quality
- **Unit Testing**: JUnit 4 with Google Truth assertions
- **Android Testing**: Robolectric for unit tests requiring Android context
- **Code Coverage**: Kover (minimum 50% for release builds)
- **Linting**: Kotlinter (ktlint wrapper) with custom Compose rules
- **CI/CD**: GitHub Actions with multi-JDK testing (17, 21, 23)

### Module Structure
```
├── app/                    # Main Android application
├── data-model/            # Shared data models and DTOs
├── service/               # Weather API service modules
│   ├── openweather/       # OpenWeatherMap API integration
│   ├── tomorrowio/        # Tomorrow.io API integration  
│   ├── openmeteo/         # OpenMeteo API integration
│   └── weatherapi/        # WeatherAPI integration
```

## Code Style & Quality Standards

### Kotlin Conventions
- Follow official Kotlin coding conventions
- Use ktlint formatting enforced by kotlinter plugin
- Compose functions should be annotated with `@Composable` and ignore naming conventions
- Prefer extension functions for utility methods
- Use data classes for immutable data structures

### Compose Guidelines
- Follow Compose best practices for state management
- Use `remember` for local state, avoid mutable state in Composables
- Implement proper state hoisting patterns
- Use Circuit's Presenter pattern for complex state logic
- Prefer stateless Composables when possible

### Dependency Injection Patterns
- Use Metro for dependency injection with Kotlin-first design
- Define scopes using `@SingleIn(AppScope::class)` for singletons
- Use `@DependencyGraph` with `@BindingContainer` objects for organizing bindings
- Implement Factory patterns for Circuit Presenters and UIs
- Use `@Multibinds` for providing sets or maps of implementations

### Metro-Specific Guidelines
- Use `@Inject` constructor injection for classes
- Create `@BindingContainer` objects to organize related bindings
- Use `@Provides` functions for complex dependency creation
- Define main app graph with `@DependencyGraph(scope = AppScope::class)`
- Use `createGraphFactory` pattern for component creation
- Test graphs should also be scoped to `AppScope::class` to access scoped bindings

### Error Handling
- Use EitherNet's ApiResult for network operations
- Handle exceptions gracefully with proper error states
- Provide meaningful error messages to users
- Log errors using Timber for debugging

## Development Workflow

### Branch Strategy
- Main branch: `main`
- Feature branches: descriptive names
- Pull requests required for all changes
- CI/CD validation on all PRs

### Build Variants
- `internal`: Development builds with debug features
- `prod`: Production builds for release

### Source Set Isolation for Build Variants
**CRITICAL**: Code in variant-specific source sets (`internal/`, `prod/`) cannot be directly referenced from the `main` source set.

#### Problem
- Classes in `app/src/internal/` are only available in `internal` variant
- Referencing them directly from `main` causes `prod` variant compilation to fail
- Example error: `Unresolved reference` when `prod` tries to compile code referencing `internal` classes

#### Solution Pattern
When adding debug-only features accessible from main code:

1. **Create variant-specific helper files** in both source sets:
   ```
   app/src/internal/java/.../FeatureHelper.kt  (real implementation)
   app/src/prod/java/.../FeatureHelper.kt      (stub/no-op implementation)
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hossain-khan/android-weather-alert](https://github.com/hossain-khan/android-weather-alert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->

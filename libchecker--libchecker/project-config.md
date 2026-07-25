---
trigger: always_on
description: LibChecker is an Android application written in Kotlin that analyzes installed applications to display information about their third-party libraries, native libraries, components (activities, services, receivers, providers), permissions, and other APK/HAP metadata. The app helps users understand the composition and architecture of Android apps on their device.
---

# GitHub Copilot Instructions for LibChecker

## Project Overview

LibChecker is an Android application written in Kotlin that analyzes installed applications to display information about their third-party libraries, native libraries, components (activities, services, receivers, providers), permissions, and other APK/HAP metadata. The app helps users understand the composition and architecture of Android apps on their device.

## Architecture & Patterns

### MVVM Architecture
- Use ViewModel classes to manage UI-related data
- Implement Repository pattern for data access
- Use LiveData/Flow for reactive data streams
- Follow Android Architecture Components best practices

### Package Structure
```
com.absinthe.libchecker/
├── annotation/          # Custom annotations
├── api/                # API-related classes, REST clients
├── app/                # Application-level components
├── compat/             # Compatibility helpers
├── constant/           # Application constants
├── data/               # Data models and entities
├── database/           # Room database components
├── features/           # Feature modules (MVVM organized)
│   ├── album/          # Snapshot comparison features
│   ├── applist/        # App listing and details
│   ├── chart/          # Statistics and charts
│   ├── home/           # Main dashboard
│   ├── settings/       # App settings
│   └── statistics/     # Statistics features
├── integrations/       # External service integrations
├── services/           # Android services
├── ui/                # Common UI components
├── utils/             # Utility classes
└── view/              # Custom views
```

### Key Libraries & Frameworks
- **Room**: Database persistence layer with DAO pattern
- **Retrofit + OkHttp**: Network requests with Moshi converter
- **Coroutines + Flow**: Asynchronous programming and reactive streams
- **Coil**: Image loading with custom transformations
- **Material Design Components**: UI framework following Material Design 3
- **ViewBinding**: Type-safe view references (no findViewById)
- **KSP**: Kotlin Symbol Processing for annotation processing
- **Protobuf**: Data serialization for efficient storage
- **Moshi**: JSON parsing with KSP code generation
- **Timber**: Structured logging
- **Firebase**: Crashlytics for crash reporting
- **Custom Libraries**: 
  - `LCRules`: Library rule matching system
  - `rulesbundle`: External rule repository integration

## Coding Conventions

### Kotlin Style
- Use ktlint for code formatting (configured in build.gradle.kts)
- Prefer `val` over `var` when possible
- Use data classes for simple data holders
- Leverage Kotlin extensions and higher-order functions
- Use nullable types appropriately with safe calls

### Android Specifics
- Use ViewBinding instead of findViewById
- Implement proper lifecycle awareness
- Handle configuration changes properly
- Use resource qualifiers for different screen sizes/orientations
- Follow Material Design guidelines

### File Naming
- Activities: `*Activity.kt` 
- Fragments: `*Fragment.kt`
- ViewModels: `*ViewModel.kt`
- Adapters: `*Adapter.kt`
- Custom Views: `*View.kt`
- Extensions: `*Extensions.kt`
- Constants: Use `object` declarations

### Resource Naming
- Layouts: `activity_*.xml`, `fragment_*.xml`, `item_*.xml`
- Drawables: `ic_*.xml` (icons), `bg_*.xml` (backgrounds)
- Colors: Use semantic naming (`colorPrimary`, `colorOnSurface`)
- Strings: Use descriptive keys with feature prefixes

## Common Development Patterns

### APK Analysis
When working with APK analysis features:
- Use `PackageUtils` for extracting package information
- Handle different ABI architectures (ARM, x86, etc.)
- Consider both APK and HAP (HarmonyOS) file formats
- Implement proper error handling for corrupted packages

### Library Detection
- Use `LCRules` for library rule matching
- Support regex patterns for dynamic library detection
- Handle both native libraries (.so files) and DEX classes
- Implement caching for performance

### UI Components
- Extend `BaseActivity` for common activity functionality
- Use `BottomSheetDialogFragment` for modal content
- Implement proper loading states and error handling
- Use RecyclerView with proper ViewHolder patterns

### Database Operations
- Use Repository pattern for database access (see `LCRepository`)
- All database operations should use Room DAOs (see `LCDao`)
- Use proper transaction handling with `@Transaction` annotations
- Handle database migrations properly in `LCDatabase`
- Use Flow for reactive database queries
- Implement proper entity relationships with foreign keys

### Networking
- Use Retrofit with coroutines for async operations
- Implement proper error handling and retry logic
- Support multiple backend endpoints (GitHub, GitLab for rules)
- Use `BaseUrlInterceptor` for dynamic URL switching
- Handle different API endpoints with proper timeouts
- Use proper HTTP caching when appropriate
- Support offline-first patterns with cached data


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LibChecker/LibChecker](https://github.com/LibChecker/LibChecker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

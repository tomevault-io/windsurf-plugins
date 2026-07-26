---
trigger: always_on
description: This document provides project-specific instructions for AI-assisted development on the Android Image Cropper library.
---

# Android Image Cropper - Claude AI Development Guide

This document provides project-specific instructions for AI-assisted development on the Android Image Cropper library.

## Project Overview

**Android Image Cropper** optimized image cropping capabilities for Android applications.

## Tech Stack

- **Language**: Kotlin 2.0.0
- **Build System**: Gradle 8.5.2 with Kotlin DSL
- **Android**: minSdk 21, compileSdk 34, targetSdk 34
- **Key Dependencies**:
  - AndroidX (AppCompat, Core, Activity, ExifInterface)
  - Kotlin Coroutines 1.8.1
  - Material Components
- **Testing**: JUnit, MockK, Robolectric, Paparazzi (snapshot testing)
- **Code Quality**: ktlint 1.3.1, Dokka (documentation)

## Module Structure

```
Android-Image-Cropper/
├── cropper/           # Main library module (published artifact)
│   └── src/
│       ├── main/      # Library source code
│       └── test/      # Unit tests
├── sample/            # Sample app demonstrating usage
│   └── src/main/      # Sample implementations
├── .github/           # CI/CD workflows
└── gradle/            # Build configuration
```

## Code Style & Conventions

### Kotlin Style
- **Code Style**: IntelliJ IDEA
- **Indentation**: 2 spaces (no tabs)
- **Continuation Indent**: 2 spaces
- **Trailing Commas**: Allowed and encouraged
- **Line Length**: No maximum (disabled)
- **Linter**: ktlint with experimental features enabled
- **Warnings**: All Kotlin warnings treated as errors

### File Organization
- Package structure: `com.canhub.cropper.*`
- Internal APIs: Classes/methods not meant for public use should be marked `internal`
- Public API: Keep minimal and well-documented
- Deprecated APIs: Mark with `@Deprecated` and provide migration path

### Code Quality Standards
1. **No unused code**: Remove unused variables, functions, and imports
2. **Type safety**: Leverage Kotlin's type system
3. **Null safety**: Use Kotlin null-safety features appropriately
4. **Coroutines**: Use for async operations (already in use for bitmap operations)
5. **Resource management**: Always close resources properly (see BitmapUtils)

## Build & Test Commands

```bash
# Build the project
./gradlew build

# Run all checks (linting, tests, etc.)
./gradlew licensee ktlint testDebug build --stacktrace

# Run unit tests
./gradlew testDebug

# Run ktlint
./gradlew ktlint

# Format code with ktlint
./gradlew ktlintFormat

# Run Paparazzi snapshot tests
./gradlew verifyPaparazziDebug

# Generate documentation
./gradlew dokkaHtml
```

## Development Workflow

### Making Changes

1. **Branch Naming**: Use developer github name `canato/` prefix (e.g., `canato/fix-rotation-bug`)
2. **Code Changes**:
   - Make changes in `cropper/` module
   - Update tests as needed
   - Update sample app if adding new features
3. **Before Committing**:
   - Run `./gradlew ktlintFormat` to auto-format
   - Run `./gradlew ktlint testDebug` to validate
   - Ensure all tests pass
4. **Commit Messages**: Follow existing style in CHANGELOG.md
   - Start with category: "API:", "Fix:", "Security:", "Technical:", etc.
   - Reference issue/PR numbers: `[#123]`

### Testing Strategy

1. **Unit Tests**: All business logic should have unit tests
   - Location: `cropper/src/test/kotlin/`
   - Use MockK for mocking
   - Use Robolectric for Android framework dependencies
2. **Snapshot Tests**: Use Paparazzi for UI component tests
3. **Sample App**: Manual testing via `sample/` module
4. **StrictMode**: Sample app has StrictMode enabled - no violations allowed

### API Changes

⚠️ **This is a published library** - API changes affect thousands of apps!

#### Breaking Changes
- **Avoid** breaking changes whenever possible
- If unavoidable:
  - Deprecate old API first
  - Provide migration guide
  - Increment major version
  - Update CHANGELOG.md with migration notes

#### Deprecation Process
1. Mark as `@Deprecated` with message and replacement
2. Keep deprecated code for at least one minor version
3. Document in CHANGELOG.md
4. Update README.md with migration guide
5. Remove only in next major version

#### Adding New APIs
1. Add to public API only if necessary
2. Document with KDoc
3. Add usage example to sample app
4. Update README.md if user-facing
5. Consider making `internal` if not needed publicly

## Release Process

**DO NOT manually trigger releases** - handled by maintainer via GitHub Actions.

1. **Snapshot Releases**: Auto-published from `main` branch to Maven Central
2. **Release Versions**: 
   - Update `VERSION_NAME` in `gradle.properties`
   - Update `CHANGELOG.md`
   - Tag release
   - GitHub Actions publishes to Maven Central

## Common Tasks

### Adding a New Feature
1. Read existing code in `cropper/src/main/kotlin/com/canhub/cropper/`
2. Understand how `CropImageView` and related classes work
3. Add feature with appropriate tests
4. Update `CropImageOptions` if adding new configuration
5. Add example to sample app
6. Update CHANGELOG.md under "In development" section
7. Update README.md if user-facing

### Fixing a Bug
1. Add a failing test that reproduces the bug
2. Fix the bug
3. Ensure test passes
4. Run full test suite
5. Update CHANGELOG.md with fix description and issue number

### Updating Dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CanHub/Android-Image-Cropper](https://github.com/CanHub/Android-Image-Cropper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

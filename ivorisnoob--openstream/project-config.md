---
trigger: always_on
description: This file is the operating manual for any agent working in this repository. Follow it strictly.
---

# OpenStream Agent Guide

This file is the operating manual for any agent working in this repository. Follow it strictly.

## Mission

OpenStream is a native Android app for discovering and streaming anime using Jetpack Compose, Material 3 Expressive, Hilt, Retrofit, Room, and Media3.

The job is not just to make code compile. The job is to keep the app coherent, expressive, maintainable, and unmistakably premium.

## Source Of Truth

When documentation conflicts with code, trust the code.

Some repo docs are stale or use the old product name `OpenStream`. Do not propagate that drift.

For new work:
- Prefer the product name `OpenStream` in code comments, docs, and user-facing copy unless the user explicitly asks for a rename.
- Verify dependencies in [gradle/libs.versions.toml](gradle/libs.versions.toml).
- Verify theme behavior in [Theme.kt](app/src/main/java/com/ivor/OpenStream/ui/theme/Theme.kt).
- Verify navigation in [AppNavigation.kt](app/src/main/java/com/ivor/OpenStream/presentation/navigation/AppNavigation.kt).
- Verify rules in [rules.md](rules.md).

## Non-Negotiable Rules

1. Use Material 3 Expressive first. If an expressive component exists, prefer it over a standard Material 3 substitute.
2. Do not invent APIs, components, or parameters. Check the actual library version before using a feature.
3. Do not create custom UI primitives that duplicate Material 3 functionality just because custom code feels easier.
4. Keep architecture boundaries intact. Presentation should not become a dumping ground for data or networking logic.
5. Do not introduce random visual styles. Every screen should feel like it belongs to the same expressive system.
6. Avoid “safe blandness”. This app should feel vivid, cinematic, and intentional, not like a default template.
7. Preserve edge-to-edge behavior and immersive layouts where the screen benefits from it.
8. If a doc says something broad and the implementation says something specific, follow the implementation.
9. Every meaningful change should leave the codebase more truthful than before. Fix stale naming and misleading comments when you touch them.
10. Compile before closing work whenever feasible.

## Tech Stack

- Kotlin 2.0.21
- Jetpack Compose
- Material 3 Expressive `1.5.0-alpha13`
- Hilt + KSP
- Retrofit + OkHttp + kotlinx serialization
- Coil 3
- Room
- Media3 ExoPlayer
- Single-activity navigation with Navigation Compose

## Current Codebase Shape

Main app entry:
- [OpenStreamApp.kt](app/src/main/java/com/ivor/OpenStream/OpenStreamApp.kt)
- [MainActivity.kt](MainActivity.kt)

Important layers:
- `data/remote`: TMDB and subtitle APIs plus DTOs
- `data/repository`: repository implementations
- `data/local`: Room database, entities, and DAOs
- `domain/repository`: repository interfaces
- `presentation`: screens, navigation, reusable components
- `ui/theme`: color, type, shape, and theme tokens

Important screens:
- Home
- Search
- Details
- Player
- Watch Later
- Downloads
- Watch History

## Architecture Rules

### Layering

- Keep API details in `data/remote`.
- Keep persistence details in `data/local`.
- Keep business-facing contracts in `domain/repository`.
- Keep orchestration in repositories and viewmodels.
- Keep composables focused on rendering state and forwarding user intent.

### ViewModels

- ViewModels own screen state.
- Prefer `StateFlow` for long-lived UI state.
- Keep UI state explicit: loading, success, error, empty, or a clear data class if multiple facets coexist.
- Do not make composables fetch directly from Retrofit, DAOs, or repositories.

### Navigation

- Routes live in [AppNavigation.kt](app/src/main/java/com/ivor/OpenStream/presentation/navigation/AppNavigation.kt).
- Reuse the existing route patterns and argument style.
- If a new screen is added, wire the route, typed arguments, and callbacks cleanly.

### Dependency Injection

- Use Hilt consistently.
- Add new versions only in [gradle/libs.versions.toml](gradle/libs.versions.toml).
- Do not hardcode versions in Gradle files.

## Material 3 Expressive Philosophy

This app should feel like anime artwork became an operating system surface.

The design target is not “clean enough”. The target is:
- bold hierarchy
- cinematic imagery
- tactile shapes
- rich surfaces
- meaningful motion
- strong containment
- obvious primary actions
- premium rhythm and spacing

### What “Expressive” Means Here

- Big text should look intentional, not merely large.
- Containers should create emphasis, not just separation.
- Motion should feel physical and directional, not decorative noise.
- Surfaces should have hierarchy. Flatness is rarely the right answer.
- Shape variation should create focus and energy without becoming chaotic.
- Artwork should do real visual work, especially on home, details, and player surfaces.

### Inspiration Anchors

When designing or refining a screen, think in terms of:
- streaming app hero moments
- anime key art as a layout driver
- editorial composition rather than plain forms on a page
- expressive motion that supports focus, not novelty
- premium Android-first craft, not generic cross-platform styling

### Design Tone

The UI should feel:
- vivid
- dramatic
- smooth
- warm
- premium
- legible

The UI must not feel:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ivorisnoob/OpenStream](https://github.com/Ivorisnoob/OpenStream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

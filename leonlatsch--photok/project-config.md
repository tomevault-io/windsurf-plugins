---
trigger: always_on
description: This document is the authoritative guide for AI agents working in the Photok codebase.
---

# AGENTS.md — Photok Codebase Guide

This document is the authoritative guide for AI agents working in the Photok codebase.  
Read it fully before writing any code.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Repository Layout](#repository-layout)
3. [Architecture](#architecture)
4. [UI Patterns](#ui-patterns)
5. [Encryption System](#encryption-system)
6. [Key Libraries](#key-libraries)
7. [Database](#database)
8. [Dependency Injection](#dependency-injection)
9. [Translations & Strings](#translations--strings)
10. [Testing](#testing)
11. [Product Flavors](#product-flavors)
12. [Rules & Conventions](#rules--conventions)

---

## Project Overview

Photok is an Android app (Kotlin) that provides an on-device encrypted photo and video vault. All media is stored inside the Android private files directory, encrypted with AES/CBC. The app supports photos, GIFs, and videos. It has no server component.

Key features: gallery, albums, backup/restore, biometric unlock, recovery phrase, password change, dark/light theme, hide-app mode (stealth dialer), and a settings screen.

---

## Repository Layout

The top-level directory contains `app/`, `gradle/`, `adr/`, `ENCRYPTION.md`, and this file. All source code lives under `app/src/main/java/dev/leonlatsch/photok/`.

To orient yourself, browse that root package — each top-level directory is a self-contained feature. The current list of features is the live source of truth; do not rely on any enumeration in this file.

All dependencies are declared in `app/build.gradle.kts` — check there for the current library stack.

Each feature follows the same internal structure:

- **`data/`** — Room DAOs, table entities, repository implementations.
- **`domain/`** — pure Kotlin interfaces, models, use cases. No Android imports.
- **`di/`** — Hilt modules that bind `data` implementations to `domain` interfaces.
- **`ui/`** — ViewModels, Fragments, Compose screens, navigator classes.
  - **`ui/compose/`** — screen-level and sub-composables.

Shared UI components and the theme live in `ui/`. Legacy base classes (`Bindable*`, `Base*`) live in `uicomponnets/`. Extensions and misc utilities live in `other/`.

---

## Architecture

### Core Pattern

The app follows a **feature-first layered architecture**:

- **`domain`** — pure Kotlin. Interfaces, models, use cases. No Android imports.
- **`data`** — Room tables, DAOs, repository implementations. Implements `domain` interfaces.
- **`di`** — Hilt modules that bind `data` implementations to `domain` interfaces.
- **`ui`** — ViewModels + Compose screens + Fragments + Navigator classes.

### Single Activity

There is a single `MainActivity` (with `DataBinding`). All screens are **Fragments** navigated via the Jetpack Navigation Component (`main_nav_graph.xml`). Fragments host Compose UIs via `ComposeView`.

### Navigation

- Declared in `main_nav_graph.xml` with Safe Args.
- Bottom-tab navigation (`MainMenu`, a Compose component) connects to top-level destinations: Gallery, Albums, Settings.
- Fragment-level navigation uses typed `Navigator` classes injected via Hilt.

---

## UI Patterns

### Compose First

**New screens must use Jetpack Compose (Material3).** Legacy screens use XML DataBinding via the `Bindable*` base classes; do not add new DataBinding screens.

### Simple MVI

Every screen follows a **simple, flat MVI**. There is no dedicated MVI framework — it is plain Kotlin + StateFlow.

**The four files per screen:**

| File | Role |
|------|------|
| `XyzFragment.kt` | `@AndroidEntryPoint Fragment`. Creates a `ComposeView`, provides `CompositionLocal`s, collects navigation event flows. |
| `XyzViewModel.kt` | `@HiltViewModel`. Exposes `val uiState: StateFlow<XyzUiState>`. Accepts events via `fun handleUiEvent(event: XyzUiEvent)`. |
| `XyzUiState.kt` | `sealed interface XyzUiState`. Common states: `Empty`, `Loading`, `Content(...)`. |
| `XyzUiEvent.kt` | `sealed interface XyzUiEvent`. One `data class`/`data object` per user action. |
| `XyzScreen.kt` | Top-level `@Composable` that takes the `ViewModel`, collects state with `collectAsStateWithLifecycle()`, and branches on the sealed state. |

Navigation events that must leave the ViewModel are sent via a `Channel<XyzNavigationEvent>` and collected in the Fragment.

**Canonical example** — `GalleryFragment` / `GalleryViewModel` / `GalleryUiState` / `GalleryUiEvent` / `GalleryScreen`.

### Legacy DataBinding Screens

Still used in `unlock` and a few others. They extend `BindableFragment<ViewDataBinding>` / `BindableActivity<ViewDataBinding>`. ViewModels can extend `ObservableViewModel` for two-way bindings. **Do not create new DataBinding screens.**

### Theme

`AppTheme` (in `ui/theme/Theme.kt`) wraps every Compose entry point. It respects the system dark/light setting. Always call `AppTheme { ... }` at the root of a Fragment's `ComposeView.setContent { }`.

### CompositionLocals

Shared objects are injected into the Compose tree via `CompositionLocal`. Check `ui/CompositionLocals.kt` and feature-specific files (e.g. `transcoding/compose/LocalEncryptedImageLoader.kt`, `settings/ui/compose/ConfigCompositionLocal.kt`) for the current set.

Provide them in the Fragment's `setContent { }` block using `CompositionLocalProvider`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonlatsch/Photok](https://github.com/leonlatsch/Photok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

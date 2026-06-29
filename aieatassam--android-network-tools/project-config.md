---
trigger: always_on
description: This file is the primary reference for Claude Code when working on this repository.
---

# CLAUDE.md – Net Swiss Knife

This file is the primary reference for Claude Code when working on this repository.
Read it in full before making any changes.

---

## Project Overview

**Net Swiss Knife** is a production-quality Android networking utilities app.
It provides a collection of networking diagnostic tools (Ping, DNS, Port Scanner, etc.)
in a clean, modern Jetpack Compose + Material 3 UI.

---

## Module Layout

```
android-network-tools/
├── app/                  # Android UI layer (Compose, ViewModels, Navigation, Hilt)
├── core-domain/          # Pure Kotlin – use cases, business logic
├── core-network/         # Pure Kotlin – network primitives, protocols, models
├── .github/
│   └── workflows/
│       ├── ci.yml               # Build & test on every push/PR to main
│       ├── release.yml          # Package & publish signed release APK/AAB
│       └── claude_add_tool.yml  # Claude-driven tool addition via workflow_dispatch
├── claude/
│   └── tool_instructions.md    # Step-by-step guide for adding new tools
├── CLAUDE.md                   # ← this file
└── README.md
```

### Layer Responsibilities

| Module | Language | Purpose |
|--------|----------|---------|
| `:core-network` | Pure Kotlin | Network interfaces, result wrappers, protocol implementations |
| `:core-domain` | Pure Kotlin | Use cases that orchestrate `:core-network` |
| `:app` | Kotlin + Android SDK | UI screens, ViewModels, navigation, DI wiring |

---

## Tech Stack

| Concern | Technology | Notes |
|---------|-----------|-------|
| Language | Kotlin 1.9.x | JDK 21, Kotlin DSL everywhere |
| UI | Jetpack Compose + Material 3 | **High-fidelity, animated UI required** |
| Navigation | Navigation Compose 2.7.x | Bottom nav + animated transitions |
| DI | Hilt 2.51.x | `@HiltViewModel`, `@AndroidEntryPoint` |
| Async | Coroutines + Flow | `viewModelScope`, `StateFlow` |
| Testing | JUnit 5 + MockK | TDD (Red → Green → Refactor) |
| Build | Gradle 8.x Kotlin DSL + Version Catalog | `gradle/libs.versions.toml` |
| Min SDK | 26 (Android 8.0) | Target SDK 34 |

---

## UI Requirements – CRITICAL

**Every screen and every tool MUST use modern, high-fidelity UI.**
Placeholder "Coming soon" screens are acceptable during development but must still
follow the patterns below. Do not ship bare `Box { Text("Coming soon") }`.

### Required Patterns

1. **Animated screen entry** – All screens must use `LaunchedEffect` + `animateFloatAsState`
   (or `AnimatedVisibility`) to animate content in on first composition.
   ```kotlin
   var visible by remember { mutableStateOf(false) }
   LaunchedEffect(Unit) { visible = true }
   AnimatedVisibility(visible, enter = fadeIn() + slideInVertically()) { ... }
   ```

2. **Navigation transitions** – `AppNavHost` must define `enterTransition`, `exitTransition`,
   `popEnterTransition`, `popExitTransition` on every `composable { }` entry.
   Use `fadeIn + slideInVertically` / `fadeOut + slideOutVertically` as defaults.

3. **Card-based layouts** – Use `ElevatedCard` or `OutlinedCard` from Material 3 for
   result panels, info sections, and tool tiles. Never use raw `Column` as a top-level container.

4. **Loading states** – Show an animated `CircularProgressIndicator` (or custom pulsing
   animation) while network operations are in progress.

5. **Shimmer / skeleton placeholders** – For items that load asynchronously, show a
   shimmer animation while loading rather than empty space.

6. **Smooth transitions** – Use `Crossfade` or `AnimatedContent` when switching between
   UI states (loading → result → error).

7. **Ripple and haptic feedback** – All interactive elements must have visible ripple
   feedback. Use `Indication` defaults from Material 3.

8. **Gradient accents** – Use `Brush.linearGradient` / `Brush.radialGradient` for hero
   areas, icons, and decorative backgrounds.

9. **Typography scale** – Use the full Material 3 typography scale:
   `displaySmall` for screen titles, `titleMedium` for section headers,
   `bodyMedium` for content, `labelSmall` for metadata.

10. **Dark-mode-safe** – All colors must reference `MaterialTheme.colorScheme.*` tokens,
    never hardcoded hex values in composables.

### Minimum UI Checklist per Tool Screen

- [ ] Animated entrance (fade + slide)
- [ ] Input field with `OutlinedTextField`, icon, and clear button
- [ ] Animated loading indicator while the use case executes
- [ ] `ElevatedCard` for displaying results
- [ ] `AnimatedContent` or `Crossfade` between states (idle / loading / success / error)
- [ ] Error state with `MaterialTheme.colorScheme.error` styling and retry action
- [ ] All strings in `strings.xml` (no hardcoded strings in composables)

---

## Development Workflow

### Adding a New Tool

Follow `claude/tool_instructions.md` exactly. Summary:

1. Write failing tests (`./gradlew :core-network:test` → RED)
2. Implement `:core-network` repository + model
3. Add `:core-domain` use case + tests
4. Create ViewModel in `:app`
5. Build the Compose screen (**follow UI requirements above**)
6. Wire into `NavRoutes` + `AppNavHost` with animated transitions
7. Add Hilt module if needed
8. Refactor while keeping tests green
9. `./gradlew test && ./gradlew :app:assembleDebug` must both pass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AieatAssam/android-network-tools](https://github.com/AieatAssam/android-network-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

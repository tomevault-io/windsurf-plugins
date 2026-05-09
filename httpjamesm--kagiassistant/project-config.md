---
trigger: always_on
description: This file documents how agentic assistants should approach the Android/Kotlin Compose project at the
---

# KagiAssistant Agent Guide

## Purpose

This file documents how agentic assistants should approach the Android/Kotlin Compose project at the
repository root. Follow these instructions before making changes and while coding to keep the
workspace consistent, safe, and maintainable.

---

## Environment & Tooling

- **Java / Kotlin**: Standard Kotlin/JVM targets bundled with Android Gradle plugin (AGP 8.x+). Use
  the Gradle wrapper located at `./gradlew` for every command.
- **Android API**: Build targets API 34 (check `app/build.gradle` for precise values). Use Android
  Studio Electric Eel+ or CLI gradlew.
- **Lint/Test Runner**: Prefer Gradle wrapper commands so that dependencies resolve reproducibly.

### Key Commands

| Purpose         | Command                                 | Notes                                                                                          |
|-----------------|-----------------------------------------|------------------------------------------------------------------------------------------------|
| Clean build     | `./gradlew clean`                       | Removes previous build outputs.                                                                |
| Full build      | `./gradlew assembleDebug`               | Main app artifact.                                                                             |
| Run lint checks | `./gradlew lint`                        | Ensures XML/Kotlin/Compose lint rules pass.                                                    |
| Format checks   | `./gradlew ktlintCheck` (if configured) | If ktlint is not configured, rely on `./gradlew lint` and `./gradlew build` as primary checks. |

---

## Repository Structure Signals

- Kotlin sources live under `app/src/main/java/space/httpjames/kagiassistantmaterial/` with Compose
  UI under `ui/` organized by screen/group (e.g., `main`, `chat`, `message`). Keep this hierarchy
  intact.
- ViewModels (e.g., `MainViewModel`) contain complex streaming/state logic; avoid moving them unless
  refactoring is part of the task.
- Resources (layouts, drawables, strings) follow Android conventions—tweak only as needed.

---

## Style & Code Guidelines

### Kotlin/Compose Formatting

1. **Imports**: Keep sorted and grouped by `androidx`/`kotlinx`/project-specific packages. Trim
   unused imports via the IDE or `./gradlew lint`. Do not reorder manually unless clearly improving
   structure.
2. **Spacing & Braces**: Use 4-space indentation. Open braces for functions/objects follow Kotlin
   conventions (`fun foo() {`). Keep expression bodies on the same line when simple (
   `fun foo() = bar`).
3. **Line Length**: Aim for <= 120 characters. Break long Compose modifier chains by placing each
   `Modifier` call on a new line.
4. **Compose Functions**: Place `@Composable` functions near related screens. Keep parameter lists
   explicit and avoid `Any` unless unavoidable.
5. **State**: Prefer immutable `StateFlow`/`MutableStateFlow` in ViewModels. Expose `asStateFlow()`
   for consumers.
6. **Builders**: When building lists (e.g., `LazyColumn`), avoid `forEach` outside Compose scope.
   Use dedicated items/lists to keep Compose recomposition predictable.

### Naming Conventions

1. **Files**: Match class names (e.g., `MainViewModel.kt` hosts `MainViewModel`). Compose screen
   files include the screen name (e.g., `ChatArea.kt`).
2. **Variables**: Use camelCase for properties and functions, PascalCase for classes/objects. Avoid
   abbreviations if unreadable (e.g., prefer `messageState` over `msgSt`).
3. **State Flows & LiveData**: Name backing flows with an underscore prefix (e.g.,
   `_messagesState`). Shared states should borrow the same suffix (e.g., `messagesState`).
4. **Composables**: Start with `@Composable fun FooBarScreen()`. When representing events (e.g.,
   `onRetryClick`), use prefix `on` + action.

### Types & Nullability

1. **Explicit Nullability**: Annotate optional values explicitly (`String?`). Ship dead-null checks
   before use.
2. **Data Classes**: Favor domain-specific data classes (e.g., `AssistantThreadMessage`). Always
   declare default values to ease serialization/deserialization.
3. **Collections**: Prefer immutable `List`/`Map` when exposing state; mutate underlying
   `MutableList` copies only within ViewModel boundaries.
4. **Coroutine Contexts**: Keep IO-bound work on `Dispatchers.IO`. Use
   `withContext(Dispatchers.Main)` only for UI-synced state updates. Avoid `runBlocking` in
   production code.

### Error Handling

1. **Exceptions**: Catch exceptions only when you can recover or provide useful logs. Bubble others
   so they surface to crash reporting.
2. **Logging**: Use simple `println`/`Log.d` sparingly—favor explicit `try/catch` with
   `e.printStackTrace()` only during debugging. Remove verbose prints before merging.
3. **Result Types**: When repository functions return `Result<T>`, inspect `isSuccess/isFailure` and
   call `exceptionOrNull()` before logging.
4. **User Feedback**: Ensure UI communicates call states (`DataFetchingState`). Update these flows
   promptly in error cases so Compose displays appropriate feedback (error screens/spinners).

### Concurrency & Coroutines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [httpjamesm/KagiAssistant](https://github.com/httpjamesm/KagiAssistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

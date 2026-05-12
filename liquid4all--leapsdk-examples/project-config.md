---
trigger: always_on
description: This repository contains example applications demonstrating the use of the LeapSDK across iOS and Android platforms.
---

# LeapSDK Examples - Agent Instructions

This repository contains example applications demonstrating the use of the LeapSDK across iOS and Android platforms.

## Table of Contents
- [Code Style & Formatting](#code-style--formatting)
- [Commit Guidelines](#commit-guidelines)
- [Project Structure](#project-structure)
- [Architecture Patterns](#architecture-patterns)
- [Concurrency & Threading](#concurrency--threading)
- [Resource Management](#resource-management)
- [Audio Handling](#audio-handling)
- [Testing](#testing)
- [Common Pitfalls](#common-pitfalls)

---

## Code Style & Formatting

### Import Rules
- **Never use star imports** (wildcard imports)
  - ❌ Bad: `import androidx.compose.material.icons.*`
  - ✅ Good: `import androidx.compose.material.icons.Icons`
  - Applies to: Kotlin, Swift, and all other languages

### Android Formatting
- Use **ktfmt with Google style** for Kotlin code
  - Add `com.ncorti.ktfmt.gradle` plugin to Android modules
  - Configure: `ktfmt { googleStyle() }`
  - Format before committing: `./gradlew ktfmtFormat`

### General Rules
- All files must end with a newline character
- **CRITICAL: Always use string resources instead of hardcoded strings in Android**
  - ❌ Bad: `Text("Recording stopped")`
  - ✅ Good: `Text(stringResource(R.string.recording_stopped))`
  - Add new entries to `res/values/strings.xml` for all user-facing text
  - This includes: UI text, error messages, accessibility descriptions, announcements
  - Exception: Debug/log messages can use string literals
- Prefer explicit types when clarity matters
- Avoid `!!` (force unwrap) - use safe calls or proper null handling

---

## Commit Guidelines

### Commit Message Format
Follow [Conventional Commits](https://www.conventionalcommits.org/):
```
<type>: <description>

[optional body explaining the change and why]

[optional breaking changes section]
```

**Types:**
- `feat:` - New feature
- `fix:` - Bug fix
- `refactor:` - Code restructuring without behavior change
- `test:` - Adding or updating tests
- `docs:` - Documentation changes
- `perf:` - Performance improvements
- `chore:` - Maintenance tasks

**Examples:**
```
feat: add graceful streaming completion for audio playback

fix: resolve race condition in audio streaming initialization

refactor: extract string provider interface for testability
```

### Commit Organization
- Create **logical commits** that group related changes
- Each commit should represent a single cohesive change
- Avoid mixing unrelated changes (e.g., bug fixes + features)
- Write detailed commit messages explaining **why**, not just **what**

---

## Project Structure

### SDK Locations
- **iOS SDK**: `~/development/leap-ios-sdk`
- **Android/KMP SDK**: `~/development/leap-android-sdk`

### iOS Examples
- Located in `iOS/` directory
- Xcode project generation via `project.yml` files
- LeapSDK integration via Swift Package Manager

### Android Examples
- Located in `Android/` directory
- Built with Gradle and Kotlin
- LeapSDK integration via Kotlin Multiplatform

### Key SDK APIs (Android)
- **Package**: `ai.liquid.leap` (not `ai.liquid.sdk`)
- **Main Classes**:
  - `LeapClient` - SDK entry point
  - `ModelRunner` - Model lifecycle management
  - `Conversation` - Chat conversation management
  - `ChatMessage` - Message representation with `Role` enum (USER, ASSISTANT, SYSTEM, TOOL)
  - `ChatMessageContent` - Sealed class: Text, Audio, Image
  - `MessageResponse` - Sealed interface: Chunk, AudioSample, Complete, etc.
  - `LeapModelDownloader` - Automatic model downloading with progress

---

## Architecture Patterns

### MVI (Model-View-Intent) Pattern
Use MVI for all Android UI state management:

**State Definition:**
```kotlin
data class MyState(
  val items: List<Item> = emptyList(),
  val isLoading: Boolean = false,
  val error: String? = null,
)
```

**Event Definition:**
```kotlin
sealed interface MyEvent {
  data object LoadData : MyEvent
  data class UpdateItem(val id: String) : MyEvent
}
```

**ViewModel Structure:**
```kotlin
class MyViewModel : ViewModel() {
  private val _state = MutableStateFlow(MyState())
  val state: StateFlow<MyState> = _state.asStateFlow()

  fun onEvent(event: MyEvent) {
    when (event) {
      is MyEvent.LoadData -> loadData()
      is MyEvent.UpdateItem -> updateItem(event.id)
    }
  }

  private fun loadData() {
    _state.update { it.copy(isLoading = true) }
    // ... implementation
  }
}
```

**Composable Usage:**
```kotlin
@Composable
fun MyScreen(state: MyState, onEvent: (MyEvent) -> Unit) {
  // UI receives state and event handler, not ViewModel
  Button(onClick = { onEvent(MyEvent.LoadData) }) {
    Text("Load")
  }
}
```

### Sealed Classes for Type Safety
- Use sealed interfaces for events, states, and responses
- Provides exhaustive `when` expressions
- Makes invalid states unrepresentable
- Example: `GenerationState.Idle`, `GenerationState.Generating`, etc.

---

## Concurrency & Threading

### Coroutine Best Practices
- **Use proper dispatchers:**
  - `Dispatchers.IO` - File I/O, network, audio operations
  - `Dispatchers.Default` - CPU-intensive work
  - `Dispatchers.Main` - UI updates (automatically in ViewModel)

- **Prefer coroutines over threads:**
  ```kotlin
  // ✅ Good

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Liquid4All/LeapSDK-Examples](https://github.com/Liquid4All/LeapSDK-Examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

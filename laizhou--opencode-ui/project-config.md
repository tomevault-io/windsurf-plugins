---
trigger: always_on
description: This repository contains the source code for the OpenCode IntelliJ Platform plugin.
---

# OpenCode IntelliJ Plugin - Developer Guide for Agents

This repository contains the source code for the OpenCode IntelliJ Platform plugin.
This document provides strict guidelines and useful context for AI agents (and human developers) working on this codebase.

## 1. Project Overview

- **Type**: IntelliJ Platform Plugin
- **Language**: Kotlin (JDK 17)
- **Build System**: Gradle (Kotlin DSL)
- **Target Platform**: IntelliJ IDEA 2024.2+ (Since Build 242)
- **Version**: 1.0.4+ (See `build.gradle.kts`)

### Core Dependencies
- **IntelliJ Platform SDK**: 2025.2.4
- **OkHttp**: 4.12.0 (Networking, SSE)
- **Gson**: 2.11.0 (JSON Serialization)

## 2. Build and Verification Commands

**CRITICAL**: Always use the provided Gradle wrapper (`./gradlew`) in the root directory.

### Build & Run
- **Build Project** (Compiles sources):
  ```bash
  ./gradlew build
  ```
- **Run IDE (Sandbox)** (Launches a separate IntelliJ instance with the plugin installed):
  ```bash
  ./gradlew runIde
  ```
- **Clean Build** (Fixes most weird cache issues):
  ```bash
  ./gradlew clean build
  ```

### Testing
- **Run All Tests**:
  ```bash
  ./gradlew test
  ```
- **Run Single Test Class** (Preferred for targeted validation):
  ```bash
  ./gradlew test --tests "ai.opencode.ide.jetbrains.util.PortFinderTest"
  ```
- **Run Single Test Method** (Precision testing):
  ```bash
  ./gradlew test --tests "ai.opencode.ide.jetbrains.util.PortFinderTest.testPortAvailability"
  ```
- **Run Tests Matching Pattern**:
  ```bash
  ./gradlew test --tests "*PortFinder*"
  ```

### Code Quality
- **Lint & Verify** (Runs detekt/ktlint if configured, plus standard checks):
  ```bash
  ./gradlew check
  ```

## 3. Code Style & Conventions

Strictly adhere to the [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html) and [IntelliJ Platform SDK Guidelines](https://plugins.jetbrains.com/docs/intellij/intro.html).

### Formatting & Naming
- **Indentation**: 4 spaces. **NO TABS**.
- **Line Length**: 120 characters preferred.
- **Classes**: `PascalCase` (e.g., `OpenCodeService`, `DiffViewerService`).
- **Functions/Properties**: `camelCase` (e.g., `findAvailablePort`, `activeSessionId`).
- **Constants**: `UPPER_SNAKE_CASE` inside `companion object`.
- **Packages**: `lowercase`, separated by dots (e.g., `ai.opencode.ide.jetbrains.util`).

### Imports
- **No Wildcards**: Avoid `import com.foo.*` unless importing >5 classes from the same package.
- **Grouping**: Group `com.intellij.*`, `java.*`, and `ai.opencode.*` separately with blank lines between groups.

### Language Policy
- **English Only**: Comments, documentation, commit messages, and user-facing strings **MUST** be in English.
- **Null Safety**: 
  - Prefer `val` over `var`.
  - Use `?.`, `?:`, and `lateinit` responsibly.
  - **AVOID `!!`** (double-bang) operators. They cause runtime NPEs. Use `checkNotNull()` or safe calls instead.

### IntelliJ Specific Patterns
- **Services**: Use `@Service(Service.Level.PROJECT)` for logic. Avoid static singletons.
- **UI Threading**: UI updates **MUST** happen on the Event Dispatch Thread (EDT).
  ```kotlin
  ApplicationManager.getApplication().invokeLater { 
      // UI updates here
  }
  ```
- **Background Tasks**: Network/IO operations **MUST** be on a background thread (e.g., `AppExecutorUtil.getAppExecutorService()`).
- **Disposal**: Services/Components holding resources must implement `Disposable`.
- **Logging**: Use `Logger.getInstance(MyClass::class.java)`. Do not use `System.out.println`.

## 4. Architecture & Patterns

The codebase follows a modular structure within the main package `ai.opencode.ide.jetbrains`:

```
src/main/kotlin/ai/opencode/ide/jetbrains/
├── OpenCodeService.kt          # CORE: Main project service, manages connection & lifecycle
├── OpenCodeToolWindow.kt       # UI: Entry point for the ToolWindow
├── QuickLaunchAction.kt        # Action: Cmd+Esc handler
├── api/                        # NETWORKING: REST & SSE clients
│   ├── OpenCodeApiClient.kt    # HTTP calls (OkHttp)
│   ├── SseEventListener.kt     # Server-Sent Events handler
│   └── models/                 # Data classes (Gson serializable)
├── diff/                       # FEATURE: Diff Viewer & Code Review
│   ├── DiffViewerService.kt    # Manages the IDE Diff Window
│   └── OpenCodeDiffEditorActions.kt # Accept/Reject logic
├── session/                    # STATE: Session & File Management
│   └── SessionManager.kt       # Tracks active session, file snapshots, and history
├── terminal/                   # INTEGRATION: Embedded Terminal
│   ├── OpenCodeTerminalVirtualFile.kt
│   ├── OpenCodeTerminalFileEditor.kt
│   └── OpenCodeTerminalLinkFilter.kt # Hyperlinks in terminal (@file:line)
├── ui/                         # UI: Dialogs & Popups
│   └── OpenCodeConnectDialog.kt
└── util/                       # UTILS: Helpers
    ├── PortFinder.kt           # Localhost port management
    └── ProcessAuthDetector.kt  # Auth token discovery
```

### Key Architectural Concepts
1.  **OpenCodeService**: The central hub. It owns the `ApiClient` and orchestrates the connection.
2.  **SessionManager**: Manages the "business logic" of the active coding session, including Git operations and file tracking.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LaiZhou/OpenCode_UI](https://github.com/LaiZhou/OpenCode_UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

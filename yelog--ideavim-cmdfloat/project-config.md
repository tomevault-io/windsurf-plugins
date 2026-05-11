---
trigger: always_on
description: This document provides comprehensive guidelines for AI agents working on the IdeaVim-CmdFloat plugin.
---

# AGENTS.md - Guidelines for AI Coding Agents

This document provides comprehensive guidelines for AI agents working on the IdeaVim-CmdFloat plugin.

## Project Overview

IdeaVim-CmdFloat is an IntelliJ Platform plugin that provides a floating command-line overlay for IdeaVim users. It intercepts `:`, `/`, and `?` triggers to display a modern overlay near the caret instead of the status bar.

- **Plugin ID**: `com.yelog.ideavim.cmdfloat`
- **Min IDE Version**: 2024.3.6 (build 231+)
- **Dependencies**: IdeaVim plugin, IntelliJ Platform
- **Language**: Kotlin (JVM 21)

## Build Commands

```bash
# Launch sandbox IDE for local debugging
./gradlew runIde

# Run all tests
./gradlew test

# Run a single test class
./gradlew test --tests "com.yelog.ideavim.cmdfloat.MyPluginTest"

# Run a single test method
./gradlew test --tests "com.yelog.ideavim.cmdfloat.MyPluginTest.testProjectService"

# Build installable plugin ZIP (output: build/distributions/)
./gradlew buildPlugin

# Full verification with Kover coverage report
./gradlew check

# Clean build artifacts
./gradlew clean

# Publish plugin (requires env vars: CERTIFICATE_CHAIN, PRIVATE_KEY, PRIVATE_KEY_PASSWORD, PUBLISH_TOKEN)
./gradlew publishPlugin
```

## Project Structure

```
src/
├── main/
│   ├── kotlin/com/yelog/ideavim/cmdfloat/
│   │   ├── actions/           # AnAction implementations (CmdfloatOverlayActions.kt)
│   │   ├── overlay/           # Core overlay UI and logic
│   │   │   ├── CmdlineOverlayManager.kt    # Main overlay lifecycle manager
│   │   │   ├── CmdlineOverlayPanel.kt      # Swing panel with input field
│   │   │   ├── CmdlineOverlayKeyDispatcher.kt  # Key event interception
│   │   │   ├── IdeaVimFacade.kt            # Reflection-based IdeaVim API access
│   │   │   ├── *Completion.kt              # Command/search completion providers
│   │   │   └── FuzzyMatcher.kt             # Fuzzy matching algorithm
│   │   ├── services/          # Project-level services
│   │   └── startup/           # Plugin initialization (ProjectActivity)
│   └── resources/
│       ├── META-INF/plugin.xml  # Plugin manifest
│       └── messages/            # i18n resource bundles
└── test/
    ├── kotlin/                  # JUnit5 + IntelliJ Platform tests
    └── testData/                # Test fixtures and snapshots
```

## Code Style Guidelines

### Formatting
- **Indentation**: 4 spaces (no tabs)
- **Max line length**: ~120 characters (soft limit)
- **Blank lines**: Single blank line between functions, two before class members
- Run `Reformat Code` and `Optimize Imports` before committing

### Imports
- Organize imports alphabetically within groups
- Group order: `java.*`, `javax.*`, `kotlin.*`, then third-party, then project
- No wildcard imports except for extension functions
- Remove unused imports

### Naming Conventions
| Element | Convention | Example |
|---------|------------|---------|
| Classes | PascalCase | `CmdlineOverlayManager` |
| Interfaces | PascalCase | `CompletionSupport` |
| Functions | camelCase | `handleTrigger()` |
| Properties | camelCase | `activeEditor` |
| Constants | SCREAMING_SNAKE | `MAX_SEARCH_WORDS` |
| Test classes | `*Test` suffix | `MyPluginTest` |
| Action IDs | `cmdfloat.*` prefix | `cmdfloat.command` |

### Kotlin Idioms
- Prefer `val` over `var` (immutability first)
- Use nullable types (`?`) instead of null checks; avoid `!!`
- Use `?.let { }`, `?.also { }`, `?:` for null handling
- Prefer `when` over chained `if-else`
- Use data classes for simple value holders
- Use `object` for singletons (e.g., `FuzzyMatcher`, `IdeaVimFacade`)
- Prefer expression bodies for single-expression functions

### Type Safety
- **Never use `as Any` or suppress type errors**
- Explicit nullability annotations on public APIs
- Use sealed classes/interfaces for closed hierarchies (e.g., `OverlayMode`, `ExpressionOutcome`)
- Prefer `runCatching { }` over try-catch for reflection calls

### Error Handling
- Use `Logger.getInstance()` for diagnostics
- Log at appropriate levels: `debug` for verbose, `info` for state changes, `warn` for recoverable errors
- Graceful degradation: if IdeaVim API changes, fall back to IDE event queue replay
- Never swallow exceptions silently; at minimum log them

## IdeaVim Integration Patterns

This plugin uses reflection to access IdeaVim internals. Key patterns:

```kotlin
// Load classes with fallback names (API evolution)
private val commandStateClass: Class<*>? = loadClass(
    "com.maddyhome.idea.vim.command.CommandState",
    "com.maddyhome.idea.vim.state.CommandState",
)

// Safe method invocation
private fun invokeBoolean(method: Method?, target: Any): Boolean? {
    if (method == null) return null
    return runCatching { method.invoke(target) as? Boolean }.getOrNull()
}

// Cache reflection lookups
private val handleKeyMethodCache = ConcurrentHashMap<Class<*>, List<Method>>()
```

## Testing Guidelines

- **Framework**: JUnit4 + IntelliJ Platform TestFramework (`BasePlatformTestCase`)
- **Test data**: Place fixtures in `src/test/testData/`
- **Naming**: Use descriptive method names like `testProjectService`, `testRename`
- **UI tests**: Use `runIdeForUiTests` Gradle task with Robot Server plugin

```kotlin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yelog/ideavim-cmdfloat](https://github.com/yelog/ideavim-cmdfloat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

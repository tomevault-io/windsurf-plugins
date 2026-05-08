---
trigger: always_on
description: Kotlin Multiplatform navigation library with Compose Multiplatform UI using a **tree-based architecture**.
---

# Quo Vadis Navigation Library

Kotlin Multiplatform navigation library with Compose Multiplatform UI using a **tree-based architecture**.

## Project Structure

| Module | Purpose | Package |
|--------|---------|---------|
| `quo-vadis-core` | Core navigation library (Navigator, NavNodes, TreeMutator) | `com.jermey.quo.vadis.core` |
| `quo-vadis-annotations` | KSP annotations (`@Stack`, `@Destination`, `@Screen`, `@Tabs`, `@Pane`, `@Modal`) | `com.jermey.quo.vadis.annotations` |
| `quo-vadis-ksp` | Code generator for zero-boilerplate navigation | - |
| `quo-vadis-gradle-plugin` | Gradle plugin for KSP configuration | - |
| `quo-vadis-core-flow-mvi` | FlowMVI integration for MVI architecture | `com.jermey.quo.vadis.flowmvi` |
| `composeApp` | Demo application showcasing all patterns | `com.jermey.navplayground.demo` |
| `feature1`, `feature2` | Multi-module feature examples | - |

## Architecture: NavNode Tree

Navigation state is an **immutable tree** of nodes:

```
StackNode (root)
├── TabNode (MainTabs)
│   ├── StackNode (HomeTab)
│   │   ├── ScreenNode (Home)
│   │   └── ScreenNode (Detail)
│   └── StackNode (ProfileTab)
│       └── ScreenNode (Profile)
└── PaneNode (adaptive layout)
    ├── StackNode (primary)
    └── StackNode (detail)
```

### Core Types

- **`NavNode`** - Base sealed interface (`quo-vadis-core/.../navigation/node/`)
- **`ScreenNode`** - Leaf node for individual screens
- **`StackNode`** - Push/pop navigation stack
- **`TabNode`** - Parallel tab-based navigation
- **`PaneNode`** - Adaptive multi-pane layouts

### Key Components

- **`Navigator`** interface - Central navigation controller (`quo-vadis-core/.../navigation/navigator/Navigator.kt`)
- **`TreeNavigator`** - Concrete implementation with StateFlow state
- **`TreeMutator`** - Pure functions for immutable tree operations (`quo-vadis-core/.../navigation/internal/tree/TreeMutator.kt`)
- **`NavigationHost`** - Compose entry point (`quo-vadis-core/.../compose/NavigationHost.kt`)

## Annotation-Based Navigation

### Defining Destinations

```kotlin
@Stack(name = "home", startDestination = Feed::class)
sealed class HomeDestination : NavDestination {
    @Destination(route = "home/feed")
    data object Feed : HomeDestination()

    @Destination(route = "home/article/{articleId}")
    data class Article(@Argument val articleId: String) : HomeDestination()
}
```

### Screen Binding

```kotlin
@Screen(HomeDestination.Feed::class)
@Composable
fun FeedScreen(navigator: Navigator) { /* ... */ }

@Screen(HomeDestination.Article::class)
@Composable
fun ArticleScreen(destination: HomeDestination.Article, navigator: Navigator) { /* ... */ }
```

### Tabs and Panes

- **`@Tabs(name)`** - Declares a tab container (no `items` array or `initialTab`). The initial tab is determined by the `@TabItem` with `isDefault = true`. If no tab is explicitly marked as default, the first discovered tab becomes the initial tab.
- **`@TabItem(parent, isDefault)`** - Child-to-parent pattern: each tab item declares which `@Tabs` it belongs to. Set `isDefault = true` on one tab to make it the initially selected tab. At most one `@TabItem` per `@Tabs` parent may have `isDefault = true`. The KSP code generator validates this at compile time and will emit errors for multiple defaults.
- **`@TabsContainer`** - Custom tab bar UI
- **`@Pane`** + **`@PaneItem`** - Adaptive multi-pane layouts
- **`@PaneContainer`** - Custom pane wrapper

**Tab Item Types:** `DESTINATION` (flat screen), `STACK` (nested navigation), `TABS` (nested tab container)

**Cross-module tabs:** Feature modules use `@TabItem(MainTabs::class)` to register as tabs — the parent `@Tabs` doesn't need to list its children. One tab should be marked `isDefault = true` to set the initial tab.

### Modal Destinations

```kotlin
@Modal
@Destination(route = "navigation_menu")
data object NavigationMenuDestination : NavDestination
```

- `@Modal` (marker, no params) — marks destination for draw-behind rendering
- Must co-exist with `@Destination`, `@Tabs`, `@Stack`, or `@Pane`
- Library renders background + modal in `Box` — user controls all visual treatment
- Dismiss via `navigator.navigateBack()` (standard stack pop)

## Build Commands

```bash
# Full build
./gradlew build

# Run all tests (Kotest)
./gradlew allTests

# Run tests
./gradlew test

# Generate coverage report (HTML)
./gradlew koverHtmlReport

# Generate coverage report (XML)
./gradlew koverXmlReport

# Verify coverage rules (when thresholds are enabled)
./gradlew koverVerify

# Detekt (static analysis)
./gradlew detekt

# Generate API docs
./gradlew :quo-vadis-core:dokkaGenerateHtml

# Run demo (platform-specific)
./gradlew :composeApp:run                           # Desktop
./gradlew :composeApp:installDebug                  # Android
```

## Code Style & Conventions

- **Destinations**: Sealed classes extending `NavDestination` with `@Stack` annotation
- **Screens**: `@Composable` functions with `@Screen(Destination::class)` annotation
- **Arguments**: Use `@Argument` on data class properties (auto-serialized for deep links)
- **Naming**: `*Destination` for sealed classes, `*Screen` for composables, `*Container` for MVI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jermeyyy/quo-vadis](https://github.com/jermeyyy/quo-vadis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->

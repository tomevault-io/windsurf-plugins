---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Scene is an Android lightweight navigation and UI composition framework based on views, designed to replace Activities and Fragments. It provides better performance (100ms+ faster than Activity startup), simpler navigation stack management, and full Jetpack Architecture Components support while maintaining Fragment compatibility.

Key characteristics:
- Single-Activity architecture with view-based navigation
- Stack-based navigation model (not navigation graph based)
- Hierarchical scene composition via GroupScene
- Full support for Lifecycle, ViewModel, SavedStateRegistry
- Modular architecture with 8 separate library modules

## Build Commands

### Building the project
```bash
./gradlew build
```

### Running all tests
```bash
./gradlew test
```

### Running tests for a specific module
```bash
./gradlew :library:scene:test
./gradlew :library:scene_navigation:test
```

### Running a single test class
```bash
./gradlew :library:scene_navigation:test --tests NavigationSceneLifecycleTests
./gradlew :library:scene:test --tests SceneLifecycleTests
```

### Installing the demo app
```bash
./gradlew installDebug
```

### Building specific modules
```bash
./gradlew :library:scene:assemble
./gradlew :library:scene_navigation:assemble
```

### Clean build
```bash
./gradlew clean build
```

## Project Structure

### Module Architecture

The project is divided into 8 library modules with clear dependencies:

1. **scene** (Base module)
   - Core Scene, GroupScene, State classes
   - Lifecycle management (6-state machine: NONE → CREATED → VIEW_CREATED → ACTIVITY_CREATED → STARTED → RESUMED)
   - Scope system for hierarchical resource management
   - No navigation features - purely scene composition and lifecycle

2. **scene_navigation** (Navigation engine)
   - NavigationScene: Stack-based navigation controller
   - NavigationSceneManager: Core navigation orchestration (1922 lines)
   - NavigationMessageQueue: Sequential operation execution
   - Push/pop operations with animation coordination
   - Scene reuse pool system for memory efficiency

3. **scene_ui** (UI templates)
   - SceneActivity: Activity wrapper to host scenes
   - AppCompatScene: Material Design support
   - SwipeBackAppCompatScene: Gesture-driven back navigation

4. **scene_navigation_compose** (Jetpack Compose integration)
   - ComposeScreen: Embed Compose UI in Scene framework
   - Lifecycle-aware recomposition

5. **scene_fragment** (Fragment bridge)
   - FragmentScene: Use Scenes within Fragment context
   - Fragment lifecycle mapping to Scene lifecycle

6. **scene_dialog** (Dialog utilities)
   - Transparent Scene-based dialog implementation

7. **scene_shared_element_animation** (Shared element transitions)
   - Scene-to-scene transition animations

8. **scene_ktx** (Kotlin extensions)
   - Extension functions for idiomatic Kotlin usage

### Key Architecture Patterns

#### Scene Lifecycle State Machine

Scenes follow a 6-state lifecycle:
```
NONE (0) → CREATED (1) → VIEW_CREATED (2) → ACTIVITY_CREATED (3) → STARTED (4) → RESUMED (5)
```

Parent-to-child synchronization ensures children reach lifecycle states after parents during entry, and before parents during exit.

#### Navigation Operation Pattern

Navigation operations (push/pop) are broken into atomic operations executed sequentially:

**Push sequence:**
1. PushCreateOperation - Create new scene
2. PushAnimationOperation - Animate in
3. PushPauseOperation - Pause previous scene
4. PushStopOperation - Stop previous scene

**Pop sequence:**
1. PopAnimationOperationV2 - Animate out
2. PopDestroyMiddlePageOperationV2 - Clean up middle pages
3. PopResumeOperation - Resume previous scene
4. PopDestroyOperation - Destroy current scene

All operations go through NavigationMessageQueue to ensure sequential, non-overlapping execution.

#### Scope System

Parallel to the scene hierarchy is a Scope hierarchy that manages:
- Scene-scoped services and dependencies
- ViewModel stores
- SavedState management
- Automatic cleanup on scope destruction

File: `library/scene/src/main/java/com/bytedance/scene/Scope.java`

#### Scene Reuse System

Optional memory optimization that allows Scene instances to be recycled:
- `IReusePool`: Interface for custom reuse pool implementations
- `DefaultReusePool`: Built-in implementation
- `ReuseBehavior`: Defines matching strategy for reusable scenes
- `NavigationReuseManager`: Orchestrates reuse lifecycle

Location: `library/scene_navigation/src/main/java/com/bytedance/scene/navigation/reuse/`

## Core Concepts

### Scene vs Fragment vs Activity

- **Scene** is analogous to Fragment but lighter and more performant
- Scenes live within a single Activity (typically SceneActivity)
- NavigationScene manages a stack of scenes similar to FragmentManager's back stack
- Key method mapping:
  - Fragment's `onCreateView()` → Scene's `onCreateView()`
  - Fragment's `onViewCreated()` → Scene's `onViewCreated()`
  - FragmentManager's `beginTransaction().replace()` → NavigationScene's `push()`

### GroupScene vs Scene

- **Scene**: Leaf component with a single view

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bytedance/scene](https://github.com/bytedance/scene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

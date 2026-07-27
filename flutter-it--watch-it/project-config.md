---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Overview

**watch_it** is a Flutter state management package built on top of get_it. It provides reactive data binding that automatically rebuilds widgets when observed data changes, eliminating the need for `ValueListenableBuilder`, `StreamBuilder`, and `FutureBuilder` widgets.

**Core philosophy**: Simple, hook-like API (similar to React Hooks/flutter_hooks) that watches registered objects in get_it and rebuilds widgets automatically.

## Development Commands

### Testing
```bash
# Run all tests
flutter test

# Run specific test file
flutter test test/watch_it_test.dart

# Run tests with coverage
flutter test --coverage
```

### Code Quality
```bash
# Analyze code
flutter analyze

# Format code (REQUIRED before commits)
dart format .

# Dry run publish check
flutter pub publish --dry-run
```

### Example App
```bash
cd example
flutter run

# Run on specific device
flutter run -d chrome
```

### Dependencies
```bash
# Get dependencies
flutter pub get

# Upgrade dependencies (check compatibility first)
flutter pub upgrade
```

## Architecture & Design Principles

### The Global State Pattern

**Critical**: watch_it uses a global variable `_activeWatchItState` (in `elements.dart`) that holds the current widget's watch state during build. This is the "magic" that allows watch functions to work without explicit parameters.

**How it works**:
1. When a widget with `WatchItMixin` or `WatchingWidget` builds, its `Element` sets `_activeWatchItState` to its local `_WatchItState` instance
2. All `watch*()` function calls access `_activeWatchItState` to register watches
3. After build completes, `_activeWatchItState` is reset to null
4. Similar pattern to `flutter_hooks` and React Hooks

**Code location**: `lib/src/elements.dart:3-32` - `_WatchItElement` mixin

### Watch Entry List & Ordering

**CRITICAL RULE**: All `watch*()` and `registerHandler*()` calls MUST:
- Be called inside `build()` method
- Be called in the SAME ORDER on every build
- Not be conditional (no `if` statements wrapping watch calls)
- Not be inside builders/callbacks

**Why**: Each watch call corresponds to a position in `_watchList` (see `watch_it_state.dart:78`). On rebuild, the counter resets and each watch call retrieves its previous `_WatchEntry` by index. Changing order breaks this mapping.

**Implementation**: `lib/src/watch_it_state.dart:138-175`
- `resetCurrentWatch()` - Resets counter to 0 at start of build
- `_getWatch()` - Retrieves watch entry by current index, increments counter
- `_appendWatch()` - Adds new watch entry when first encountered

### Widget Integration

Three ways to use watch_it:

1. **WatchingWidget** (extends StatelessWidget) - `lib/src/widgets.dart`
2. **WatchingStatefulWidget** (extends StatefulWidget) - `lib/src/widgets.dart`
3. **Mixins**: `WatchItMixin` or `WatchItStatefulWidgetMixin` - `lib/src/mixins.dart`

All create custom `Element` subclasses (`_StatelessWatchItElement` or `_StatefulWatchItElement`) that:
- Initialize `_WatchItState` on mount
- Set/unset `_activeWatchItState` around build
- Dispose watch entries on unmount

### Data Types & Watch Functions

**Hierarchy**:
```
Listenable (base)
├─ ChangeNotifier
└─ ValueListenable<T>
   └─ ValueNotifier<T>
```

**Watch function mapping**:
- `watch()` - Any `Listenable` (ChangeNotifier, ValueNotifier, etc.)
- `watchIt()` - `Listenable` from get_it
- `watchValue()` - `ValueListenable` property from get_it object
- `watchPropertyValue()` - Property of `Listenable`, rebuilds only when property value changes
- `watchStream()` - `Stream<T>`, returns `AsyncSnapshot<T>`
- `watchFuture()` - `Future<T>`, returns `AsyncSnapshot<T>`

**Implementation**: All in `lib/src/watch_it.dart`

### Handler Pattern (Side Effects)

Handlers execute side effects (show dialogs, navigation, etc.) instead of rebuilding:
- `registerHandler()` - For `ValueListenable` changes
- `registerChangeNotifierHandler()` - For `ChangeNotifier` changes
- `registerStreamHandler()` - For `Stream` events
- `registerFutureHandler()` - For `Future` completion

**Key difference**: Handlers receive a `cancel()` function to unsubscribe from inside the handler.

### Lifecycle Helpers

- `createOnce()` - Create objects on first build, auto-dispose on widget destroy
- `createOnceAsync()` - Async version, returns `AsyncSnapshot<T>`
- `callOnce()` - Execute function once on first build
- `onDispose()` - Register dispose callback
- `pushScope()` - Push get_it scope tied to widget lifecycle

**Use case**: Creating `TextEditingController`, `AnimationController`, etc. in stateless widgets

### Tracing & Debugging

Two-level tracing system:

1. **Widget-level**: Call `enableTracing()` at start of build
2. **Subtree-level**: Wrap with `WatchItSubTreeTraceControl` widget

**Performance consideration**: Subtree tracing only active if `enableSubTreeTracing = true` globally (checked in `_checkSubTreeTracing()`)

**Custom logging**: Override `watchItLogFunction` to integrate with analytics/monitoring

**Events tracked**: `WatchItEvent` enum in `watch_it_tracing.dart` - rebuild, handler, createOnce, scopePush, etc.

## Common Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flutter-it/watch_it](https://github.com/flutter-it/watch_it) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

---
trigger: always_on
description: `inview_notifier_list` is a Flutter package that builds a `ListView` or `CustomScrollView` and notifies when child widgets enter or leave a defined viewport area. Used for auto-playing videos on scroll, lazy-loading content, triggering animations, etc.
---

# CLAUDE.md — Project Guide for Claude Code

## What This Package Does

`inview_notifier_list` is a Flutter package that builds a `ListView` or `CustomScrollView` and notifies when child widgets enter or leave a defined viewport area. Used for auto-playing videos on scroll, lazy-loading content, triggering animations, etc.

**pub.dev:** https://pub.dev/packages/inview_notifier_list
**496+ likes, 6.7k weekly downloads.**

## Architecture

The package is intentionally small — 5 source files, ~400 lines of Dart:

```
lib/src/
├── inview_notifier.dart          # Base StatefulWidget — scroll listener, throttled stream, viewport detection
├── inview_notifier_list.dart     # InViewNotifierList (ListView), InViewNotifierCustomScrollView, InViewNotifierWidget
├── inview_state.dart             # ChangeNotifier — tracks which widget ids are currently in-view
├── inherited_inview_widget.dart  # InheritedWidget — passes InViewState down the tree
└── widget_data.dart              # Simple data class — stores BuildContext + id pairs
```

### How It Works (Core Flow)

1. `InViewNotifier` wraps a `ScrollView` in a `NotificationListener<ScrollNotification>`
2. Scroll events are throttled via `stream_transform`'s `audit()` into a `StreamController`
3. On each throttled event, `InViewState.onScroll()` iterates all registered widget contexts
4. For each widget, it calculates `deltaTop` and `deltaBottom` relative to the viewport
5. The user-provided `IsInViewPortCondition` function evaluates whether the widget is "in view"
6. `InViewState` extends `ChangeNotifier` — it calls `notifyListeners()` only when state actually changes
7. `InViewNotifierWidget` uses `AnimatedBuilder` on the `InViewState` to rebuild when visibility changes

### Key Design Decisions

- **Throttling over debouncing:** `audit()` emits the last event at the end of each period, so detection stays responsive during fast scroll without flooding the handler.
- **No widget-level configuration:** The `IsInViewPortCondition` is set once at the list level, not per-widget. This keeps the API simple.
- **Builder pattern over children list:** Uses `ListView.builder` under the hood for lazy construction.
- **`ChangeNotifier` dedup:** Lines 86-95 of `inview_state.dart` prevent redundant `notifyListeners()` calls when a widget's in-view state hasn't actually changed. Do not remove these guards.

## Development

### Setup

```bash
flutter pub get
cd example && flutter pub get && cd ..
```

### Quality Checks

```bash
dart format --set-exit-if-changed .   # Formatting
dart analyze --fatal-infos            # Zero warnings, zero infos
flutter test                          # All tests must pass
flutter test --coverage               # Must stay above 90%
```

### Running the Example App

```bash
cd example
flutter run
```

The example has 4 tabs: basic in-view detection, expanded detection area, auto-play video, and CustomScrollView with grids + lists.

## Conventions

### Dart Style

- **Dart 3+ required.** SDK constraint: `>=3.0.0 <4.0.0`
- Use `super.key` and super parameters — never `Key? key` with `super(key: key)`
- Use modern `typedef` syntax: `typedef Foo = void Function(int)` not `typedef void Foo(int)`
- Use switch expressions where appropriate — no `late` + `switch`/`break` pattern
- Prefer `const` constructors
- No unnecessary `Container` wrappers — use `SizedBox`, `ColoredBox`, or return the child directly
- No `!` operator on Flutter APIs that return non-nullable in Flutter 3.x (e.g., `RenderAbstractViewport.of()`)

### Testing

- **CI enforces 90% minimum coverage.** PRs below this fail.
- Every test must catch a real bug. No coverage padding.
- Do NOT test Flutter framework behavior (e.g., `ChangeNotifier.addListener` works). Test YOUR logic.
- Do NOT test `toString()` or debug helpers.
- Tests are self-contained — do NOT import from `example/lib/`. Inline test widgets in the test file.
- Use `binding.setSurfaceSize()` inside `testWidgets` callbacks, never in `setUp`.

### What Not to Do

- Do not add features without an issue discussion first. The API surface is intentionally small.
- Do not refactor code you didn't change.
- Do not add comments, docstrings, or type annotations to unchanged code.
- Do not "improve" surrounding code when fixing a bug.
- Do not wrap things in `Container` just to add a single property.

### Commits

```
fix: description     # Bug fixes
feat: description    # New features
test: description    # Test changes
chore: description   # Maintenance (deps, CI, docs)
docs: description    # Documentation
ci: description      # CI/CD changes
```

### Changelog

User-facing changes go in `CHANGELOG.md`. Use `## [Unreleased]` for pending work. Maintainer assigns version on release.

## File Map

| File | What It Does | Touch With Care |
|------|-------------|-----------------|
| `lib/src/inview_state.dart` | Core algorithm — viewport intersection math | YES — the scroll detection logic is battle-tested |
| `lib/src/inview_notifier.dart` | Scroll listener + throttled stream | YES — stream lifecycle is subtle |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rvamsikrishna/inview_notifier_list](https://github.com/rvamsikrishna/inview_notifier_list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

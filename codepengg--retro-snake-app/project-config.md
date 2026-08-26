---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Read this first

A detailed knowledge base lives in **`.ai/`**. Start with **`.ai/memory.md`** (the
condensed operating context), then the relevant `.ai/context/*.md` for whichever
layer you are touching. `.ai/planning/tasks.md` holds the prioritised backlog with
analysis already done — check it before reporting a "new" issue.

⚠️ **`README.md` is partly stale.** It documents a `20x20` grid as static consts in
`game_state.dart` (the grid is now constructor-injected and device-dependent),
lists sound files that do not exist (`eat.wav`, `game_over.wav`, `start.wav`), and
tells you to uncomment the `fonts:` block in `pubspec.yaml` (already done). Trust
the code and `.ai/` over the README.

## Commands

```bash
flutter pub get

flutter analyze                                  # must stay at "No issues found"
flutter test                                     # 30 tests
flutter test test/game_state_test.dart           # single file
flutter test --plain-name "rejects a 180-degree reversal"   # single test by name

flutter run -d <device>
flutter build apk --debug
flutter build apk --release       # requires android/key.properties (untracked)
flutter build appbundle --release # Play Store artifact
```

`flutter` is a shell function wrapping fvm. It prints harmless
`_fvm_project_root_flutter: command not found` lines before real output — ignore
them.

## Architecture

Layered-by-type widget app, ~2,650 lines in `lib/`. **No state-management package,
no DI container, no networking, no database, no router package, no CI.** These are
deliberate decisions recorded in `.ai/decisions/adr-001..003.md`, appropriate for a
single-feature offline game. Do not add any of them without approval.

```
lib/
├── main.dart              MaterialApp · ThemeData · Routes constants
├── game/
│   ├── game_state.dart    DOMAIN — the only clean boundary
│   ├── snake_game.dart    loop + input + audio + lifecycle + layout
│   └── snake_painter.dart Canvas rendering
└── ui/
    ├── app_colors.dart    the only design-token file
    ├── start_screen.dart  (755 lines)
    ├── game_over_screen.dart (688 lines)
    └── common/retro_button.dart  the only extracted shared widget
```

### The one hard rule

**`lib/game/game_state.dart`'s entire import list is `import 'dart:math';`.** No
Flutter, no plugins, no I/O. That property is why 23 unit tests run without a
widget tree, and it is the only real architectural seam in the project. If you
need change notification, wrap it — `GameController extends ChangeNotifier` owning
a plain `GameState` — rather than importing Flutter into the domain file.

Business logic belongs in `GameState`. Widgets orchestrate; they do not compute
game rules.

### Data flow

`Timer.periodic(200ms)` → `GameState.update()` → `setState(() {})` → rebuild →
new `SnakePainter`. Direction input is buffered one tick: `setDirection` writes
`_nextDirection`, `update()` commits it to `_direction`. The 180°-reversal guard
compares against the **committed** direction, which is what prevents a two-step
self-kill.

### Painter contract

`GameState` is mutated in place, so a painter comparing it by reference always
reports "unchanged". `SnakePainter` snapshots value fields in its constructor and
compares those in `shouldRepaint`, plus `super(repaint: animation)` so animations
repaint without `setState`. **Any painter reading `GameState` must copy this
pattern.**

### Persistence

One key: `high_score` (`int`) in `SharedPreferences`. That is the entire persisted
state of the app.

**`GameOverScreen` is the single writer.** `SnakeGame` used to write it too, which
made the "is this a new best?" comparison always false and left the ★ NEW HIGH
SCORE ★ banner structurally unreachable. Three widget tests enforce this — adding a
second writer will fail them, intentionally. New-best is **strictly greater**; a
tie does not celebrate.

### Navigation

`Navigator` 1.0 with three flat named routes. Use the `Routes` constants in
`main.dart`, never string literals. **Return home with
`popUntil(ModalRoute.withName(Routes.home))`**, never `pushReplacementNamed('/')` —
the latter stacks a second `StartScreen` and leaks its three `AnimationController`s,
timer, and `AudioPlayer` once per round.

## Deliberate behaviours — do NOT "fix" these

Each has a source comment. Changing any is a product decision, not a bug fix.

1. **Entering the tail cell is fatal**, though it is vacated that same tick.
   Diverges from classic Snake.
2. **Resume does not auto-unpause** — the player may not be looking at the screen.
3. **`dispose()` is the sole owner of `AudioPlayer` lifetime.** Lifecycle callbacks
   call `stop()`, never `dispose()`; disposing on background previously left the
   player unusable and double-disposed it.
4. **Random geometry is generated once in `initState`** into a `late final` field,
   never in `build` or an `AnimatedBuilder` — `GameOverScreen`'s 1 Hz countdown
   `setState` would reshuffle it. Regression-tested.
5. **`_endGame()` delays navigation 2 s (death) / 1 s (win)** so the crash sound
   and glitch animation play. Navigation must never move back into `build()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codepengg/retro_snake_app](https://github.com/codepengg/retro_snake_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Flutter Tic-Tac-Toe app — Human vs CPU, local play only. Technical challenge following Clean Architecture principles.

## Commands

```bash
flutter pub get                            # Install dependencies (from root)
flutter test packages/game                 # Run game tests
flutter test packages/login                # Run login tests
flutter test packages/game/test/usecases/play_game_test.dart  # Single test file
flutter run                                # Run app (from apps/tic_tac_toe/)
flutter analyze apps/tic_tac_toe           # Analyze the app
flutter analyze packages/game              # Analyze a package
```

## Architecture

Clean Architecture (Ports & Adapters) with four layers inside each feature package (e.g. `packages/game/`):

- **domain/** — Business logic, models, exceptions, strategies. No external dependencies. All models use `Equatable` for value equality. Pure Dart only.
- **application/** — Use cases (one class per action: `StartGame`, `PlayGame`, `PlayIa`). Each use case orchestrates domain logic. Contains `ports/` defining abstract interfaces (e.g. `GameRepository`) that the infrastructure layer implements.
- **infrastructure/** — Implementation details: `persistence/` for repository implementations (e.g. `inmemory/InMemoryGameRepository`), `configuration/` for Riverpod provider wiring (DI).
- **presentation/** — Flutter UI: `view/` for pages, `viewmodel/` for Riverpod AsyncNotifiers (MVI pattern), `widgets/` grouped by domain concept (`board/`, `cell/`, `game/`).

Shared utilities live in `packages/core/` (e.g., `groupBy` extension on `Iterable`).

**Dependency rule**: domain ← application ← infrastructure / presentation. Inner layers never import outer layers. Infrastructure implements application ports.

## Domain Model

- **Game** — Abstract root aggregate holding board, difficulty, and state. Concrete subtypes represent game states: `PlayerTurnGame`, `IaTurnGame`, `HasWinnerGame`, `DrawGame`.
- **Board** — 3×3 grid of `Cell` objects. Win detection via `WinningCondition.hasWinner()`.
- **Cell** — Value object with `x`, `y`, and `Symbol`.
- **Symbol** — Enum: `x`, `o`, `empty`.
- **Difficulty** — Enum: `easy`, `medium`, `difficult`.

## Testing

Tests are in `packages/game/test/` mirroring the source structure:
- `usecases/` — Use case tests (start game, play game, play IA)
- `winning_conditions/` — Board win detection (rows, columns, diagonals)
- `strategy/` — IA strategy tests (random, blocking, minimax)
- `model/` — Game state transition tests
- `presentation/view/` — Page widget tests
- `presentation/viewmodel/` — Notifier tests
- `presentation/widgets/` — Widget tests (board, cell, game)
- `builder/game_builder.dart` — Test builder pattern for constructing `Game` instances

Tests use Given-When-Then structure with descriptive ASCII board representations in test names. Prefer fakes and stubs over mocks.

### Widget Testing Patterns

**Parameterized tests with Dart 3 records:**
```dart
final scenarios = [
  (GameStatusViewModel.playerTurn, 'Your turn'),
  (GameStatusViewModel.victory, 'Victory!'),
];

for (final (status, expectedMessage) in scenarios) {
  testWidgets('Should display $expectedMessage', (tester) async { ... });
}
```

**Faking providers:**
```dart
// For FutureProviders: use overrideWith
startGameUseCaseProvider.overrideWith((ref) async => FakeStartGame())

// For sync providers: use overrideWithValue
analyticsProvider.overrideWithValue(const NoAnalytics())
```

**Faking use cases:**
```dart
class _FakeStartGame implements StartGame {
  @override
  Future<Game> execute(StartGameCommand command) async =>
      PlayerTurnGame(board: Board.generate3x3(), difficulty: command.difficulty);
}
```

**Testing navigation with `ref.listen()`:**
- `ref.listen()` only fires on state *changes*, not the initial state.
- To test navigation, simulate a state change (e.g., `idle` → `gameStarted`).

**Scrolling to off-screen elements:**
```dart
await tester.scrollUntilVisible(find.text('Play'), 100);
await tester.tap(find.text('Play'));
```

**Testing `Visibility` widget:**
```dart
final visibility = tester.widget<Visibility>(find.byType(Visibility));
expect(visibility.visible, isFalse);
```

**Use domain objects in tests** instead of `@visibleForTesting` constructors:
```dart
BoardViewModel createBoard({List<(int x, int y, Symbol symbol)> plays = const []}) {
  var board = Board.generate3x3();
  for (final (x, y, symbol) in plays) {
    board = board.playAt(x, y, symbol);
  }
  return BoardViewModel.from(board);
}
```

## Flutter Rules

The following rules are adapted from the [official Flutter AI rules](https://github.com/flutter/flutter/blob/main/docs/rules/rules.md).

### Dart Style

- Follow [Effective Dart](https://dart.dev/effective-dart).
- Use `PascalCase` for classes, `camelCase` for members/variables/functions/enums, `snake_case` for files.
- Lines should be 80 characters or fewer.
- Write concise, modern Dart code. Prefer functional and declarative patterns.
- Prefer immutable data structures. Widgets (especially `StatelessWidget`) should be immutable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/c-leffy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

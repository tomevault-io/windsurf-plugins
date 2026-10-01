---
trigger: always_on
description: Tests use **TUnit** (not xUnit/NUnit). Key differences from other frameworks:
---

# Testing Guidelines

Tests use **TUnit** (not xUnit/NUnit). Key differences from other frameworks:

- Assertions are async: `await Assert.That(value).IsEqualTo(expected)`
- Use `.IsTrue()`/`.IsFalse()` for booleans, not `.IsEqualTo(true)` (analyzer TUnitAssertions0015)
- Use `.IsNull()` for null checks, not `.IsEqualTo(null)` (analyzer TUnitAssertions0014)
- Test runner uses `Microsoft.Testing.Platform` (configured in `global.json`)

## Test conventions

- **Naming**: `MethodName_Condition_ExpectedResult` (e.g. `ActivateAsync_SetsIsActive`, `Constructor_NullConfig_Throws`)
- **Test doubles**: Hand-written stubs and fakes (e.g. `TestScreen : Screen`, `StubConductor : IConductor`, `SimpleServiceProvider : IServiceProvider`). No mocking library is used.
- **One concept per test**: Each test verifies a single expectation with clear Arrange/Act/Assert structure.
- **Test class separation**: Separate classes for distinct responsibilities. Example: `ViewModelLocatorTests` tests runtime locator behavior, `ViewModelLocatorConfigurationTests` tests the mapping configuration API.

## TUnit assertion pitfalls

- **`IsEquivalentTo` is order-insensitive** — do not use it to verify event ordering. Use indexed assertions instead:
  ```csharp
  // WRONG: does not verify order
  await Assert.That(events).IsEquivalentTo(["PropertyChanging", "PropertyChanged"]);

  // CORRECT: verifies exact order
  await Assert.That(events).Count().IsEqualTo(2);
  await Assert.That(events[0]).IsEqualTo("PropertyChanging");
  await Assert.That(events[1]).IsEqualTo("PropertyChanged");
  ```
- **`IsEqualTo` fails on mismatched collection types** (e.g. `List<string>` vs `string[]`). Use indexed assertions or ensure types match.

## Test anti-patterns

- **`Task.Delay` for synchronization** — Never use `await Task.Delay()` to wait for UI events. Use event-driven `TaskCompletionSource` with `.WaitAsync()` timeout instead.
- **Relying on `[ModuleInitializer]` side effects** — module initializers are lazy: they run on first load of the platform module, and the TUnit host does not guarantee that load has happened before a test body. Do not write a test whose setup depends on registration having occurred, and do not "fix" that with a `[Before(Test)]` guard that silently re-registers, which makes the test pass even if the production `[ModuleInitializer]` is deleted. Test the component directly instead, via `InternalsVisibleTo`.
- **Keyless `[NotInParallel]`** — a keyless constraint makes the test run completely alone, so it cannot overlap anything, including classes that share no key. A **class-level** keyed `[NotInParallel("key")]` is different: it serializes that class's own test methods and also holds it apart from other classes carrying the same key, but it does nothing against classes that do not carry the key. Prefer a class-level key when several classes must stay in step, and keyless on a single test when one test alone must be isolated. Per the TUnit docs, keyless is the most restrictive option, so reach for it only when a key is genuinely insufficient.
- **Testing concurrency on non-thread-safe types** — `Conductor<T>` and MVVM types are not thread-safe. Test sequential behavior, not concurrency.
- **Test name doesn't match assertion** — Name must describe what is verified, not what is set up.
- **GC tests without a positive case** — Verify both dead handlers are removed AND live handlers still work. Applies to all edge-case/cleanup tests.
- **Duplicate tests across files** — Each test class owns a clear responsibility. Don't place the same behavioral test in two files (e.g. PopupLifecycle tests should only be in `PopupLifecycleTests.cs`, not also in `WindowLifecycleTests.cs`).

## Test patterns

- **Static helpers over base classes** — Use private static helper methods (e.g. `CreateDialogWithXamlRoot`, `OpenPopupAsync`) for shared test setup rather than inheritance hierarchies.
- **Tuple returns for fixtures needing cleanup** — When a helper creates multiple objects the test must dispose, return a tuple: `(ContentDialog dialog, Window window)`. This makes cleanup explicit and visible.
- **Cross-platform test alignment** — The same behavioral tests should exist across WPF, Avalonia, and WinUI for shared features. When a platform can't run a test (e.g. Avalonia Popup in headless mode), document the limitation and ensure coverage on the other platforms. Platform-specific features (e.g. `ContentDialogLifecycle`) only need tests on the platform that owns them.
- **Verify both paths in edge-case tests** — For cleanup, error-path, and GC tests, verify both the failure/cleanup path and the normal/live path.

## Parallel execution and `[NotInParallel]`

Static state shared across test classes requires `[NotInParallel("key")]` at class level to prevent race conditions. Always write the key as a **string literal, never `nameof(...)`**: a key is a coordination token shared by every class that touches the same state, and `nameof` binds it to one class's name. `"StaticExecutingEvent"` spans three classes (`AsyncDelegateCommandTests`, `EventAggregatorTests`, `WeakStaticEventHandlerTests`), so `nameof` there would have produced three different keys and silently disabled the isolation. Named keys in use:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tibel/Caliburn.Light](https://github.com/tibel/Caliburn.Light) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

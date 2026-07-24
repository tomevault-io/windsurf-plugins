---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project DNA

These values act as a filter for every proposed change. Reject anything that violates them.

- **Zero dependencies** — no runtime npm dependencies in library code (see `doc/decisions/0003-zero-dependencies.md`). Refuse any change that adds an `import` from an external package inside `lib/` or `bin/`.
- **No metaprogramming** — no Proxies, no `Object.defineProperty`, no `__proto__` assignment. Every property access must follow the normal prototype chain. Every line must be readable in a classroom without prior explanation.
- **OOP in the Smalltalk spirit** — behaviour through message sends and polymorphism, not conditionals over types. Add a method to an object before adding a `switch`/`if` chain in a caller.

## Commands

```bash
npm test                          # run all tests (self-tests the framework)
npm test tests/path/file_test.js  # run a single test file
npm run lint                      # ESLint check
npm run lint:fix                  # ESLint auto-fix
npm run playground:reset          # copy template → tests/playground_test.js
npm run playground:run            # run playground file
npm run playground:clear          # delete playground file
npm run test:coverage             # coverage report via c8 (see reports/coverage/)
npm run test:mutation             # mutation testing via Stryker (slow, see reports/mutation/)
```

Node 22+ is required. The repo uses asdf; `.tool-versions` pins `nodejs 22.21.0`.

## Architecture

### Execution flow

`bin/testy_cli.js` → `ScriptAction.for(params)` (handles `-h`, `-v`, or `RunTests`) → `Testy#run()` → loads test files via dynamic `import()` → `TestRunner#run()` → emits callbacks → `ConsoleUI` → `Formatter`.

### Core domain (`lib/core/`)

- **`TestRunner`** — orchestrates suites; aggregates counts; drives the `onFinish`/`onSuccess`/`onFailure` callbacks.
- **`TestSuite`** — owns a list of `Test` instances; runs each; fires `onStart`/`onFinish` suite callbacks.
- **`Test`** — executes one test body; stores a `TestResult`; fires `whenSuccess`/`whenFailed`/`whenErrored`/`whenPending`/`whenSkipped` callbacks.
- **`Asserter` / `Assertion`** — assertion DSL. Primary form: `assert.that(actual).isEqualTo(expected)`. Shorthand form: `assert.areEqual(actual, expected)`. Custom failure message: `assert.withDescription('msg').isTrue(expr)`.

### UI layer (`lib/ui/`)

**`ConsoleUI`** is the event hub. It converts `TestRunner`/`TestSuite`/`Test` callbacks into calls on the active `Formatter`.

**`Formatter`** (base class) defines the full event protocol as no-op methods. Concrete subclasses:
- `ConsoleFormatter` — human-readable, coloured output (default)
- `TapFormatter` — TAP version 13 (streaming, one line per test)
- `JsonFormatter` — single JSON document emitted on `displayRunnerEnd`

**`FormatterFactory.for(output, console, i18n)`** maps the `output` config key to the right class; unknown values fall back to `ConsoleFormatter`.

### Configuration (`lib/config/`)

`Configuration` merges `.testyrc.json` (user, optional) over `default_configuration.json`. Keys: `directory`, `filter`, `language`, `failFast`, `randomOrder`, `timeoutMs`, `output`.

`ParametersParser` maps CLI flags to a config object. Paired flags (`-l`, `-d`, `-e`, `-o`) are sanitized into `"flag value"` strings before dispatch to per-flag parser classes.

### i18n (`lib/i18n/`)

`translations.json` holds all UI strings in `en`, `es`, `it`, `pt`. Failure messages implement `.expressedIn(i18n)` so they resolve lazily in the configured language. Adding a translation key requires entries in **all four** language sections — `tests/core/translation_keys_consistency_test.js` will catch any missing ones.

### Test support (`tests/support/`)

- **`tests_factory.js`** — factories for `Test` instances in each result state (`aPassingTest`, `aFailingTest`, `anErroredTest`, `aPendingTest`, `anExplicitlySkippedTest`). Factories accept an `asserter` argument.
- **`formatter_helpers.js`** — `runResultsWith(suiteName, ...factories)` runs tests and returns `{ runner, suite }`; `driveFormatter(formatter, runner, suite)` replays the full event stream into a formatter.
- **`runner_helpers.js` / `suites_factory.js`** — lower-level helpers for building runners and suites in tests.

## Simplicity Guardian

A CI check that blocks PRs violating Testy's simplicity contract. It runs on every PR against `main`.
See `doc/decisions/0017-simplicity-guardian.md` for the full decision record.

**Run locally:**
```bash
node bin/simplicity-guardian.js          # text output, exits 1 on violations
node bin/simplicity-guardian.js --format json   # machine-readable output
```

**Three layers checked:**
- **Zero-dependency** — no external package imports in `lib/` or `bin/`
- **Metaprogramming** — no `new Proxy`, `Object.defineProperty`, or `__proto__=`
- **Fan-out** — no file with more than 7 imports

**ESLint note:** `class-methods-use-this` is intentionally absent from Testy's ESLint config.
In Smalltalk-style OOP, methods are polymorphic message handlers — a method that doesn't reference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ngarbezza/testy](https://github.com/ngarbezza/testy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

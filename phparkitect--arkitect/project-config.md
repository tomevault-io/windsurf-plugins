---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

The project requires **PHP `^8.0`** (`composer.json`); CI/dev pins **8.1** (`.php-version.dist`). PHP 8 features (constructor promotion, union types, trailing commas) are used freely — there is no PHP 7.x compatibility requirement.

## What this is

PHPArkitect is a static-analysis CLI that enforces architectural rules over a PHP codebase. Users write rules as PHP in a `phparkitect.php` config file (e.g. "classes in `App\Domain` must not depend on `App\Infrastructure`") and run `phparkitect check` in CI. It parses source into an AST with `nikic/php-parser`, never executing user code.

It is a public **open-source** library (`phparkitect/phparkitect` on Packagist, MIT), installed as a dev dependency in thousands of projects and run mostly in CI.

## Priority: user-facing developer experience

The people who use this tool are developers writing rules and reading failures, so DX is a first-class concern, not an afterthought — weigh it in every change:
- **Violation output is the product.** When a rule fails, the message (class, broken rule, the `because(...)` reason) must make the cause obvious. Keep it actionable and don't regress its clarity.
- **The config DSL is a public API.** The fluent `Rule`/`Expression`/`Architecture` API and expression class names are how users express intent — favour readable, discoverable, backwards-compatible naming. Breaking it breaks every consumer's `phparkitect.php`.
- **Document user-facing changes** in `README.md` and `docs/rules.md` in the same change; a new rule nobody can find isn't done.
- **Fail gracefully.** Confusing errors, crashes on valid PHP, or noisy output are DX bugs — treat them as such.

## Commands

```bash
make build      # full build: composer install + csfix + psalm + phpunit
make test       # run all tests (bin/phpunit)
make csfix      # apply php-cs-fixer (run before committing)
make psalm      # static analysis (psalm errorLevel 2)
make phar       # build phparkitect.phar via box
```

Run a single test by name — the Makefile has pattern targets:

```bash
make test_SomeTest        # → bin/phpunit --filter test_SomeTest
make SomeTest             # → bin/phpunit --filter SomeTest
bin/phpunit --filter testMethodName
bin/phpunit tests/Unit/Analyzer/FileVisitorTest.php
```

Before committing, `make csfix && make psalm && make test` must all pass (CI gates merges on a green build).

## Pipeline (how a `check` run flows)

`bin/phparkitect` → `PhpArkitectApplication` registers the commands in `src/CLI/Command/` (`Check`, `GenerateBaseline`, `Init`, `DebugExpression`). Options shared between analysis-running commands (`--config`, `--target-php-version`, `--autoload`, `--ignore-baseline-linenumbers`) are defined once in `CommonOptions`, and the shared console plumbing (process limits, progress, heading, timing) in `CommandRuntime` — both composed by each command.

`Check::execute` (`src/CLI/Command/Check.php`) parses the CLI options into a `CheckOptions` value object and delegates to `CheckHandler` (`src/CLI/CheckHandler.php`), the application service that runs the actual flow:
1. `ConfigBuilder` loads the user's `phparkitect.php`, which receives a `Config` and registers `ClassSet`s + `Rule`s.
2. `Runner` (`src/CLI/Runner.php`) iterates every `SplFileInfo` in each `ClassSet`, parses it via `FileParser`, and gets back `ClassDescription`s.
3. Each `Rule` is evaluated against each `ClassDescription`, accumulating `Violations`.
4. `Baseline` (`src/CLI/Baseline.php`) filters out known/ignored violations.
5. A `Printer` (`text` / `json` / `gitlab`, `src/CLI/Printer/`) renders the result; exit code is non-zero if violations remain.

## Architecture — the two halves

**Analysis (`src/Analyzer/`)** turns source into a queryable model. `FileParser` runs `nikic/php-parser` with `FileVisitor`, a `NodeVisitorAbstract` that walks the AST and builds a `ClassDescription` (name, dependencies, interfaces, parents, traits, attributes, docblock types, final/abstract/readonly/enum flags…). Dependencies are collected from every construct that references another type: `new`, `instanceof`, static calls/constants, typed properties, param/return types, etc. `DocblockParser` (via `phpstan/phpdoc-parser`) extracts types from annotations. **Most parser bugs are a missing `handle*` case in `FileVisitor` for some syntax — add the case and a fixture-based test.**

**Rules / Expressions (`src/Rules/`, `src/Expression/`)** are the constraint model:
- An **`Expression`** (`src/Expression/Expression.php`) is one composable boolean check over a `ClassDescription`, producing a `Violation` with a description when it fails. All concrete checks live in `src/Expression/ForClasses/` (e.g. `ResideInOneOfTheseNamespaces`, `NotHaveDependencyOutsideNamespace`, `HaveNameMatching`, `Extend`, `Implement`, `IsFinal`, `HaveAttribute`). Every class here implements `Expression` — that invariant is itself enforced by `phparkitect.php`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phparkitect/arkitect](https://github.com/phparkitect/arkitect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

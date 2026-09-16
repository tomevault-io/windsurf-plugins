---
trigger: always_on
description: handles its own concerns inline - argument parsing, its own `try`/`catch`
---

# AGENTS.md

This file provides guidance to AI agents when working with
code in this repository.


## Project Overview

This project is a PHP library for building panel-based terminal forms:
keyboard-driven questionnaires that collect answers interactively through a
TUI or headlessly from a JSON payload and environment variables.


## PHP Application Architecture


### Library API

This package is a library consumed programmatically - it has no CLI entry point
of its own. The public surface is:

- **`DrevOps\PhpTui\Tui`** - the facade: collect a form's answers, headlessly or
  through the interactive panel TUI.
- **`DrevOps\PhpTui\Builder\Form`** - the fluent builder for declaring a form's
  panels and fields.

A consumer declares a form with `Form::create(...)->panel(...)` and drives it
through the `Tui` facade.

The facade also exposes **primitives** - standalone, theme-drawn elements that
collect no answer and never run inside the panel:

- `Tui::progress()` (`DrevOps\PhpTui\Primitive\Progress`) wraps a slow callback
  with a spinner (unknown length) or a determinate bar (known total) for work
  that runs around the form.
- `Tui::output()` (`DrevOps\PhpTui\Primitive\Output`) draws the static chrome
  around a form: boxes and cards, aligned tables, the five status lines of
  `DrevOps\PhpTui\Primitive\Status`, definition lists, wrapped text, rules and a
  banner.

Every piece a primitive draws routes through a `render*()` method on the theme
that takes only plain strings and arrays - never a `Field`, `Panel` or
`Answers`. `renderCard()` is the single renderer behind both the standalone
card and the one a markup block draws in a panel, grid included, so a theme
override restyles the two together. Keep it that way: a renderer that reaches
for form state cannot be used standalone.

A border is not a card. Anything occupying a rectangle - the screen, a region,
any block - declares its edges with the border capability, and the renderer
sizes the box while the theme draws the glyphs. A block never learns the space
it was given, so it declares its edges and never draws them.


### Namespace Structure

- Source code: `DrevOps\PhpTui\`
- Tests: `DrevOps\PhpTui\Tests\`
- Autoloading: PSR-4 via Composer

## Commands

### Code Quality

```bash
# Run all linters (PHPCS, PHPStan, Rector)
composer lint

# Auto-fix code style issues
composer lint-fix

# Individual tools
./vendor/bin/phpcs # Check coding standards
./vendor/bin/phpcbf # Fix coding standards
./vendor/bin/phpstan # Static analysis (level 9)
./vendor/bin/rector --dry-run # Check Rector suggestions
```

### Testing

```bash
# Run all PHPUnit tests (fast, no coverage)
composer test

# Run with coverage reports
composer test-coverage
# Coverage reports: .logs/.coverage-html/index.html, .logs/cobertura.xml

# Run specific test file
./vendor/bin/phpunit tests/phpunit/Unit/TuiTest.php

# Run specific test method
./vendor/bin/phpunit --filter testMethodName
```


### Dependencies


```bash
# Clean and reinstall dependencies
composer reset # removes vendor/ and composer.lock
composer install
```

## Code Quality Standards

### Three-Layer Quality Stack

1. **PHP_CodeSniffer** - Drupal coding standards + strict types requirement
  - Config: `phpcs.xml`
  - Rules: Drupal standard, Generic.PHP.RequireStrictTypes
  - Relaxed rules in test files (long arrays, missing function docs)

2. **PHPStan** - Level 9 static analysis
  - Config: `phpstan.neon`
  - Ignores: Untyped iterables in tests/data providers

3. **Rector** - PHP 8.3 modernization + code quality
  - Config: `rector.php`
  - Sets: PHP_83, CODE_QUALITY, CODING_STYLE, DEAD_CODE,
    TYPE_DECLARATION

### Coding Conventions

- All PHP files must declare `strict_types=1`
- Use single quotes for strings (double quotes if containing single quote)
- All files must end with a newline character
- Local variables/method arguments: `snake_case`
- Method names/class properties: `camelCase`
- **A method that answers a yes/no question about state is named `is*`.** The
  prefix is what marks a return as boolean, so a reader never has to open the
  method to find out - `isRequired()`, `isMultiple()`, `isScrolling()`,
  `isSelectable()`, `isQueryDriven()`, `isUnicode()`, `isGhost()`. There is no
  `has*` form: possession is state, so `has*` and `is*` were one group and
  `is*` is the one spelling.

  Two things are not state predicates and keep their own names:

  - A **command that reports its own outcome**. Its job is to do something and
    its boolean says whether that happened - `accept()`, `capture()`,
    `activate()`, `load()`, `leave()`, `prepare()`. An `is` prefix would
    misname the work. A method that both acts and answers is a command.
  - A **lookup taking what it is asked about** - `Answers::has(string $id)`,
    `Key::is(KeyName $name)`, `Bounds::contains($value)`. These ask about an
    argument rather than about the object's own state, so they read as verbs.
- **Never model a closed set of values as string literals.** Any value that is
  one-of-a-fixed-set (a kind, a state, a mode, a source) is a backed or pure
  enum, and every property, parameter and return that carries it is typed with
  the enum - existing examples: `FieldType`, `Provenance`, `Source`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drevops/phptui](https://github.com/drevops/phptui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->

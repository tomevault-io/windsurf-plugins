---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`nette/phpstan-rules` is a PHPStan extension package for Nette library developers. It provides custom rules and type extensions used when analysing Nette libraries with PHPStan. The package is consumed by individual Nette repositories via their PHPStan configuration.

## Commands

```bash
composer phpstan          # Run static analysis (level 8)
composer tester           # Run all tests
vendor/bin/tester tests/SomeTest.phpt -s   # Run a single test
```

## Architecture

- **`src/`** — Extension source code, PSR-4 autoloaded under `Nette\PHPStan\` namespace
- **`src/Tester/TypeAssert.php`** — Reusable type inference testing helper for Nette Tester (used by other Nette packages)
- **`extension.neon`** — Entry point, includes `extension-php.neon` and `extension-nette.neon`, auto-included by `phpstan/extension-installer`
- **`extension-php.neon`** — Generic PHP-level extensions (RemoveFailingReturnType, ClosureTypeCheckIgnore)
- **`extension-nette.neon`** — All Nette package extensions (component-model, forms, schema, tester, utils), separated by comments
- **`phpstan.neon`** — Self-analysis config (level 8, analyses `src/` and `tests/`)

### How extensions are registered

Each extension class is registered as a service in NEON with the appropriate tag. Common tags:
- `phpstan.rules.rule` — custom rules
- `phpstan.collector` — collectors
- `phpstan.broker.expressionTypeResolverExtension` — expression type resolution (runs before all dynamic extensions)
- `phpstan.broker.dynamicFunctionReturnTypeExtension` — dynamic function return types
- `phpstan.broker.dynamicMethodReturnTypeExtension` — dynamic instance method return types
- `phpstan.broker.dynamicStaticMethodReturnTypeExtension` — dynamic static method return types
- `phpstan.ignoreErrorExtension` — conditional error suppression
- `phpstan.broker.propertiesClassReflectionExtension` — magic properties
- `phpstan.broker.methodsClassReflectionExtension` — magic methods
- `phpstan.broker.typeSpecifyingExtension` — type narrowing

### Namespace conventions

Extensions for specific Nette packages use dedicated namespaces: `Nette\PHPStan\ComponentModel\` for nette/component-model, `Nette\PHPStan\Schema\` for nette/schema, `Nette\PHPStan\Utils\` for nette/utils, future packages follow the same pattern (`Nette\PHPStan\Forms\`, `Nette\PHPStan\Application\`, etc.). Generic PHP-level extensions use `Nette\PHPStan\Php\`.

### ExpectArrayReturnTypeExtension

`ExpectArrayReturnTypeExtension` (`DynamicStaticMethodReturnTypeExtension`) narrows the return type of `Expect::array()` from `Structure|Type` to `Structure` or `Type`. It inspects the argument: no argument, null, empty array, or non-Schema values → `Type`; all values implementing `Schema` → `Structure`; mixed/unknown → fallback to declared union. Config: `extension-nette.neon`.

### ArrowFunctionVoidIgnoreExtension

`ArrowFunctionVoidIgnoreExtension` (`IgnoreErrorExtension`) suppresses `argument.type` when an arrow function (which always returns a value) is passed to a parameter typed as `Closure(): void`. The list of affected functions/methods is configurable via a flat NEON list — plain names for functions (`testException`), `Class::method` notation for methods (`Tester\Assert::exception`). Config: `extension-nette.neon`.

### ClosureTypeCheckIgnoreExtension

`ClosureTypeCheckIgnoreExtension` (`IgnoreErrorExtension`) suppresses `expr.resultUnused` for the runtime type validation pattern `(function(Type ...$p) {})(...$args)`. Config: `extension-php.neon`.

### RemoveFailingReturnTypeExtension

`RemoveFailingReturnTypeExtension` (`ExpressionTypeResolverExtension`) removes `|false` or `|null` from return types of native PHP functions and methods where the error return value is trivial or outdated. It handles `FuncCall`, `MethodCall`, and `StaticCall` in a single class. Configuration uses a flat list in NEON — plain names for functions (`json_encode`), `Class::method` notation for methods (`Normalizer::normalize`). It runs before all `DynamicReturnTypeExtension` implementations, delegates to them via `DynamicReturnTypeExtensionRegistry`, and strips `|false` from the result. For `preg_replace`, `preg_replace_callback`, `preg_replace_callback_array`, and `preg_filter` it strips `|null` instead (these return null on PCRE error). For `preg_replace_callback_array` pattern validation checks array keys. Config: `extension-php.neon`.

### FalseToNullReturnTypeExtension

`FalseToNullReturnTypeExtension` (`DynamicStaticMethodReturnTypeExtension`) narrows the return type of `Helpers::falseToNull()` from `mixed`. It removes `false` from the argument type and adds `null` — e.g. `string|false` → `string|null`, `false` → `null`, types without `false` pass through unchanged. Config: `extension-nette.neon`.

### StringsRegexHelper (Utils)

`StringsRegexHelper` consolidates the regex logic shared by the `Nette\Utils\Strings` extensions below, to avoid the flag mapping drifting across them. It has two kinds of members:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nette/phpstan-rules](https://github.com/nette/phpstan-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->

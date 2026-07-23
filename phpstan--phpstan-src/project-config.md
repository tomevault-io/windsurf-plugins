---
trigger: always_on
description: PHPStan finds bugs in PHP code without running it. It analyses the entire codebase and reports type errors, undefined methods/properties/variables, dead code, incorrect PHPDoc types, and many other issues. It understands PHP's type system deeply, including generics, union/intersection types, literal types, conditional return types, and template types expressed through PHPDocs.
---

# PHPStan - PHP Static Analysis Tool

PHPStan finds bugs in PHP code without running it. It analyses the entire codebase and reports type errors, undefined methods/properties/variables, dead code, incorrect PHPDoc types, and many other issues. It understands PHP's type system deeply, including generics, union/intersection types, literal types, conditional return types, and template types expressed through PHPDocs.

## Key concepts

- **Rule levels 0-10**: Incremental adoption from basic checks (level 0) to strict mixed type enforcement (level 10). Levels are cumulative. Level 0 covers unknown classes/functions/methods and undefined variables. Level 5 checks argument types. Level 6 requires typehints. Level 9 enforces explicit `mixed`. Level 10 enforces implicit `mixed`.
- **Baseline**: Allows adopting higher rule levels by recording existing errors in a baseline file, so only new errors are reported.
- **Bleeding edge**: Preview of next major version features, shipped in current stable release via `bleedingEdge.neon`.
- **Result cache**: PHPStan caches analysis results and only re-analyses changed files and their dependents.
- **Parallel analysis**: Files are analysed in parallel across multiple child processes using React PHP.
- **Configuration**: NEON format (Nette configuration). Main config is `phpstan.neon`, level configs in `conf/config.level*.neon`, services in `conf/services.neon`.

## Running PHPStan

```bash
# Analyse with a specific level
vendor/bin/phpstan analyse -l 8 src tests

# Clear result cache
vendor/bin/phpstan clear-result-cache

# Generate baseline
vendor/bin/phpstan analyse --generate-baseline

# Debug mode (shows which files are being analysed)
vendor/bin/phpstan analyse --debug
```

## Running tests

```bash
make tests
```

Rules are tested using `PHPStan\Testing\RuleTestCase`, type extensions with `PHPStan\Testing\TypeInferenceTestCase`.

## Architecture

The codebase lives under `src/` with PSR-4 autoloading mapping `PHPStan\` to `src/`. Key architectural components:

### PHPStan\Analyser - Core analysis engine

The analysis pipeline: `Analyser` orchestrates `FileAnalyser`, which uses `NodeScopeResolver` to walk the AST and invoke registered `Rule` implementations.

**`NodeScopeResolver`** (`src/Analyser/NodeScopeResolver.php`) - The central engine of PHPStan. It traverses the AST (from nikic/php-parser) and maintains a `MutatingScope` that gets updated at each node. It handles:
- Control flow analysis (if/else, switch, try/catch, loops, match)
- Variable assignments and type narrowing
- Function/method call resolution
- Closure and arrow function scoping
- Type narrowing from conditions (`instanceof`, `is_*()`, `===`, etc.)
- PHPDoc type resolution and `@var`/`@param`/`@return` processing
- Invoking registered Rules and Collectors at each AST node

**`MutatingScope`** (`src/Analyser/MutatingScope.php`) - Holds the current state of analysis after each AST node. It tracks:
- Variable types (assigned, possibly-defined, narrowed)
- Current context: namespace, class, method/function, trait, anonymous function
- Property types and initialization state
- Constant values
- Expression types via `getType(Expr $expr): Type`
- Native types vs PHPDoc types

The `Scope` interface (`src/Analyser/Scope.php`) is the public API that rules and extensions use. `MutatingScope` is the internal implementation.

**`TypeSpecifier`** (`src/Analyser/TypeSpecifier.php`) - Narrows types based on conditions. When NodeScopeResolver processes an `if ($x instanceof Foo)`, TypeSpecifier determines that `$x` is `Foo` in the truthy branch and `not Foo` in the falsy branch. It uses `TypeSpecifierContext` (truthy/falsey/true/false/null) to track which branch is being processed. Extensible via `FunctionTypeSpecifyingExtension`, `MethodTypeSpecifyingExtension`, and `StaticMethodTypeSpecifyingExtension`.

**`ExpressionTypeHolder`** (`src/Analyser/ExpressionTypeHolder.php`) - Stores an expression's type together with a `TrinaryLogic` certainty (Yes = definitely this type, Maybe = possibly this type). This is the building block of how `MutatingScope` tracks variable types - it maps expression keys to `ExpressionTypeHolder` instances.

**`ConstantResolver`** (`src/Analyser/ConstantResolver.php`) - Resolves PHP constant names to their types. Handles 150+ predefined PHP constants with PHP-version-aware types (e.g. different return types based on `PHP_VERSION_ID`). Also resolves user-configured constant type mappings.

**`InitializerExprTypeResolver`** (`src/Reflection/InitializerExprTypeResolver.php`) - Resolves types of constant expressions and initializers (default parameter values, property defaults, constant values). Handles arithmetic, casts, function calls, and binary operations in constant contexts where a full Scope is not available.

### Analysis pipeline in detail

1. `AnalyseCommand` parses CLI arguments, builds the DI container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phpstan/phpstan-src](https://github.com/phpstan/phpstan-src) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

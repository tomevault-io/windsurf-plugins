---
trigger: always_on
description: PHP code formatter that parses source into custom token objects, applies transformation rules, assembles tokens into lines, splits long lines, and renders the result. Uses PHP's native `PhpToken::tokenize()` internally.
---

# AGENTS.md

## Project Overview

PHP code formatter that parses source into custom token objects, applies transformation rules, assembles tokens into lines, splits long lines, and renders the result. Uses PHP's native `PhpToken::tokenize()` internally.

## Key Commands

- `composer check` — tests, then PHPStan, then style check, in order (fails fast). **Run this to verify all changes.**
- `composer test` — PHPUnit tests only
- `composer analyze` — PHPStan static analysis only
- `composer cs-check` — check code style using this project's own formatter
- `composer cs-fix` — apply code style fixes
- `composer cs-preview` — show styled output without modifying files
- `composer test-coverage` — generate HTML coverage report to `tmp/coverage/`

## Gotchas

- Do not run `composer cs-fix` on in-flight code; fix style issues manually.
- New test files at the `tests/` top level must be added to `php-styler.php` or `composer cs-check` won't cover them.
- `src/Token/` has ~550 files — use grep/search, not directory listing.

## Architecture

### Pipeline

`Styler::__invoke(string $code): string` runs 6 stages:

1. **Parse** (`Parser`) — tokenizes PHP into `AToken` objects, tracking nesting and paren depth. The Parser also handles modifier ordering and missing-visibility insertion directly during tokenization (via `handleModifier()`, `atClassBody()`, `hasPrevVisibility()`), and always adds control structure braces. Token classes can detect abstract property hooks via nesting (`atNesting()`) and parsed-token inspection (`hasPrev()`, `getParsedAt()`), and reorder hook tokens at the closing brace (`swapParsedAt()`).
2. **TokenRules** — `TokenRule` implementations transform the token array (e.g., `NormalizeImports`, `NormalizeTypeOrder`)
3. **Assemble** (`Assembler`) — groups tokens into `Line` objects by indent level
4. **Split** (`Splitter`) — enforces line-length limits via prioritized split points; runs `normalizeIndents` twice (before and after expansion)
5. **LineRules** — `LineRule` implementations transform lines (e.g., `MergeParenBrace`, `NormalizeTrailingCommas`)
6. **Render** — each `Line` renders its tokens to string with indentation

### Key Directories

- `src/Token/` — `AToken` subclasses in a flat directory (no subdirs, ~550 files: 519 concrete `T*` tokens, 30 `A*` interfaces/abstracts, and the `DocblockParsing` trait used by the 16 comment-token classes to share the `getDocblock()` method). Includes whitespace tokens, split-point markers (`TSplit` subclasses with priority), and marker interfaces for classification. Many of the simpler token classes inherit `AToken::parse()` without overriding — a dedicated `parse()` is only needed when the token has contextual dispatch logic beyond "add self and optional space-after."
- `src/Rule/` — 27 concrete rule implementations. TokenRules (22): `CollapseEmptyBody`, `ConvertFromYodaConditions`, `ConvertToShortArraySyntax`, `ConvertToShortListSyntax`, `ConvertToYodaConditions`, `ConvertVarToPublic`, `ExpandConstDeclarations`, `ExpandGroupedImports`, `ExpandPropertyDeclarations`, `InsertNewParens`, `InsertPublicVisibility`, `MergeParenBracket`, `NormalizeImports`, `NormalizeModifierOrder`, `NormalizeTypeOrder`, `RemoveBom`, `RemoveEmptyAnonymousClassParens`, `RemoveEmptyAttributeParens`, `RemoveLanguageConstructParens`, `RemoveLeadingWhitespace`, `RemovePhpClosingTag`, `RemoveRepeatedSemicolons`. LineRules (5): `MergeParenBrace`, `NormalizeMemberOrder`, `NormalizeMemberSpacing`, `NormalizeTrailingCommas`, `RemoveTrailingBlankLines`. Configured in Format classes as `[RuleClass::class => [args]]`.
- `src/Format/` — `AFormat` interface → `PlainFormat` (base with styles) → `DeclarationFormat` (opinionated defaults: next-line braces, lower keywords, default rules). Vendor formats in `src/Format/Vendor/` (`DoctrineFormat`, `Percs30Format`, `SymfonyFormat`).
- `src/Parallel/` — `WorkerPool` (spawns child processes via `proc_open` for parallel file processing) and `WorkerResult` (per-file result value object).
- `src/Command/` — CLI commands (`Apply`, `Check`, `Diff`, `Preview`) with corresponding `*Options` classes, plus an internal `Worker` command for parallel execution. Dispatched via `AutoShell\Console` from `bin/php-styler`. The `apply`, `check`, and `diff` commands accept `--workers=N` (or `auto`) to process files in parallel.

### Configuration

`php-styler.php` at project root returns a `Config` object specifying `files` and `format`. The `files` list explicitly enumerates which paths are style-checked — `src/`, `tests/Rule/`, `tests/Token/`, and individual test files. `tests/Examples/` and `tests/Format/` are not style-checked.

## Coding Conventions

- PHP 8.4+ required; uses PHP 8.4 features (asymmetric visibility, interface property hooks)
- PSR-4 autoloading: `PhpStyler\` → `src/`
- 4-space indentation, LF line endings
- PHPStan level max (covers `src/` and `tests/`, excludes `tests/Examples/`)
- The project formats its own source code with its own tool
- CI runs on Linux and Windows, PHP 8.4 only (`.github/workflows/ci.yml`)

## Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pmjones/php-styler](https://github.com/pmjones/php-styler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

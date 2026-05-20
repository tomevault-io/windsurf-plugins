---
trigger: always_on
description: Welcome, AI assistant. Please follow these guidelines when contributing to this repository.
---

# AI Contribution Guidelines

Welcome, AI assistant. Please follow these guidelines when contributing to this repository.

## Project Overview

This is a demo application to showcase the main features of the EasyAdminBundle project, a third-party Symfony bundle for creating admin backends.
It's also a learning tool for developers who want to understand how to use EasyAdminBundle in their own Symfony applications.

**Requirements:** PHP 8.4+, Symfony 8.x

## General Rules

- Language: American English for code, comments, commits, branches
- Code quotes: wrap strings with single quotes in PHP, CSS, JavaScript
- Text quotes: straight quotes only (`'` and `"`, no typographic)
- Security: prevent XSS, CSRF, injections, auth bypass, open redirects

### Do Not Edit
- `vendor/` - managed by Composer
- `var/` - Symfony cache/logs
- `composer.lock`, `yarn.lock` - update only via package manager commands

## Commands

### Development
```bash
composer install  # Install PHP dependencies
```

### Pre-Commit Checklist

Before submitting changes, run these commands to verify them:

If PHP code changed:
- [ ] `php-cs-fixer fix --dry-run` shows no issues
- [ ] Run tests with:
  ```bash
  ./vendor/bin/simple-phpunit                    # All tests
  ./vendor/bin/simple-phpunit tests/Field/       # Specific directory
  ./vendor/bin/simple-phpunit --filter=testName  # Specific test
  ```

If Twig templates changed:
- [ ] `./vendor/bin/twig-cs-fixer lint templates/` passes

If translations changed:
- [ ] all locales updated consistently; use English as placeholder if unsure

## Git and Pull Requests

### Commit Messages
- Use imperative mood: "Add feature" not "Added feature"
- First line: concise summary (50 chars max)
- Reference issues when applicable: "Fix #123"
- No period at end of subject line

### Branch Naming
- Feature: `<short description>` (e.g., `add_new_field_type`)
- Bug fix: `fix_<issue number>` (e.g., `fix_123`)
- Use lowercase with underscores

## PHP Code Standards

### Syntax and Style
- PHP 8.4+ syntax with constructor property promotion
- PSR-1, PSR-2, PSR-4, PSR-12 standards
- Yoda conditions: `if (null === $value)` (project convention)
- Strict comparisons only (`===`, `!==`)
- Braces required for all control structures
- Trailing commas in multi-line arrays
- Blank line before `return` (unless only statement in block)

### Naming
- Variables/methods: `camelCase`
- Config/routes/Twig: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- Classes and enum cases: `UpperCamelCase`
- Abstract classes: `Abstract*` (except test cases)
- Interfaces: `*Interface`, Traits: `*Trait`, Exceptions: `*Exception`
- Most classes add a suffix showing its type:
  `*Controller`, `*Configurator`, `*Context`, `*Dto`, `*Event`,
  `*Field`, `*Filter`, `*Subscriber`, `*Type`, `*Test`
- Templates/assets: `snake_case` (e.g., `detail_page.html.twig`)

### Class Organization
1. Properties before methods
2. Constructor first, then `setUp()`/`tearDown()` in tests
3. Method order: public, protected, private

### Code Practices
- Don't add `declare(strict_types=1);` to PHP files
- Use enums instead of constants for fixed sets of values
- Avoid `else`/`elseif` after return/throw
- Use `sprintf()` for exception messages with `get_debug_type()` for class names
- Exception messages: capital letter start, period end, no backticks
- `return null;` for nullable, `return;` for void
- Always use parentheses when instantiating: `new Foo()`
- Add `void` return types on test methods
- Use service autowiring (don't configure explicitly in `config/services.php`)
- Comments: only for complex/unintuitive code, lowercase start, no period end
- Error messages: concise but precise and actionable (e.g. include class names, file paths)
- Handle exceptions explicitly (no silent catches)
- Config files in PHP format (`translations/*.php`)
- Use admin pretty URLs instead of generating them with `AdminUrlGenerator`

### PHPDoc
- No `@return` for void methods
- No single-line docblocks
- Group annotations by type
- `null` last in union types

## Templates (Twig)

- Modern HTML5 and Twig syntax
- Icons: FontAwesome 6.x names
- All user-facing text via `|trans` filter (no hardcoded strings)
- Translation logic in templates, not PHP (use `TranslatableInterface`)
- Use Twig components from EasyAdmin when possible (`<twig:ea:* />`)
- Accessibility: `aria-*` attributes, semantic tags, labels
- When adding links, use `path()` and admin pretty URLs instead of building them with `ea_url()`

## JavaScript

- ES6+ syntax
- 4-space indentation
- `camelCase` for variables and functions

## CSS

- Standard CSS only (no SCSS/LESS)
- 4-space indentation
- Bootstrap 5.3 classes and utilities
- Don't use nested rules
- Logical properties: `margin-block-end` instead of `margin-bottom`
- `kebab-case` for class names
- Responsive design required; use only these Bootstrap breakpoints:
    - Medium (md): ≥768px
    - Large (lg): ≥992px
    - Extra large (xl): ≥1200px

## Testing

- Only write functional tests (no unit tests) unless strictly necessary
- Only test features and behavior of this app, not Symfony or EasyAdmin internals

### Writing Tests
- Extend `WebTestCase` for functional tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EasyCorp/easyadmin-demo](https://github.com/EasyCorp/easyadmin-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

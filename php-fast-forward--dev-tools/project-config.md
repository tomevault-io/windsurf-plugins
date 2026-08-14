---
trigger: always_on
description: FastForward\DevTools is a Composer plugin that aggregates multiple PHP development tools into a single unified command. It provides automated execution of tests, static analysis, code styling, refactoring, and documentation generation.
---

# AGENTS.md

## Project Overview

FastForward\DevTools is a Composer plugin that aggregates multiple PHP development tools into a single unified command. It provides automated execution of tests, static analysis, code styling, refactoring, and documentation generation.

**Key Technologies:**

- PHP 8.3+
- Composer plugin
- PHPUnit 12.5 (testing)
- Rector 2.x (automated refactoring)
- Easy Coding Standard (code style)
- phpDocumentor (API docs)
- GrumPHP (Git hooks)
- PHPSpec/Prophecy (test doubles)

## Setup Commands

```bash
# Install as development dependency
composer require --dev fast-forward/dev-tools:dev-main

# Install dependencies
composer install
```

## Development Workflow

```bash
# Run all standard checks (refactoring, code styling, docs, tests, reports)
composer dev-tools

# Automatically fix code standards issues
composer dev-tools:fix

# Individual commands
composer dev-tools tests         # Run PHPUnit tests
composer dev-tools code-style    # Check and fix code style (ECS + Composer Normalize)
composer dev-tools refactor      # Refactor code using Rector
composer dev-tools phpdoc        # Check and fix PHPDoc comments
composer dev-tools docs          # Generate HTML API documentation
composer dev-tools wiki          # Generate Markdown documentation for wiki
composer dev-tools reports       # Generate docs frontpage and reports
composer dev-tools agents        # Sync packaged project agents into .agents/agents
composer dev-tools:sync          # Sync managed repository assets and packaged agent surfaces
```

**Notable Specialized Commands:**

- `composer skills`: Synchronize packaged skills into consumer `.agents/skills`
- `composer funding`: Sync funding metadata between `composer.json` and `.github/FUNDING.yml`
- `composer codeowners`: Generate `.github/CODEOWNERS` from repository metadata
- `composer gitattributes`: Manage export-ignore rules in `.gitattributes`
- `composer gitignore`: Synchronize managed `.gitignore` content
- `composer license`: Generate or refresh repository `LICENSE` files
- `composer dependencies`: Run dependency analysis workflows
- `composer metrics`: Generate PhpMetrics reports and related artifacts
- `composer update-composer-json`: Normalize managed `composer.json` settings
- `composer changelog:entry|check|next-version|promote|show`: Manage changelog-driven release workflows
- `composer dev-tools:sync --dry-run|--check|--interactive`: Preview managed-file drift while intentionally skipping wiki, skills, and agents

## Testing Instructions

```bash
# Run all tests with coverage
composer dev-tools tests

# Run tests matching a pattern
composer dev-tools tests -- --filter=CodeStyle

# Run with coverage report
composer dev-tools tests -- --coverage=.dev-tools/coverage
```

**Test Configuration:**

- PHPUnit XML config: `phpunit.xml`
- Test namespace: `FastForward\DevTools\Tests\`
- Source namespace: `FastForward\DevTools\`
- Coverage required (strict metadata enforcement)
- Coverage threshold: configured in `phpunit.xml`

**Test Patterns:**

- Tests use PHPUnit 12.x with Prophecy for mocking
- Custom PHPUnit extension: `FastForward\DevTools\PhpUnit\Runner\Extension\DevToolsExtension`

**Creating/Updating Tests:**

- Use skill `phpunit-tests` in `.agents/skills/phpunit-tests/` for creating or updating PHPUnit tests with Prophecy
- Run skill when: creating new test classes, adding test methods, or fixing existing tests

## Code Style

**PHP Coding Standard:** PSR-12 with Symfony style

**ECS Configuration:** `ecs.php`

- Uses Symfony, PSR-12, and Symplify rule sets
- PHPDoc alignment: left-aligned
- PHPUnit test case static methods: use `self::`
- Skips: vendor, resources, public, tmp directories

**File Organization:**

- Source code: `src/` (PSR-4: `FastForward\DevTools\`)
- Tests: `tests/` (PSR-4: `FastForward\DevTools\Tests\`)
- Commands: `src/Console/Command/`
- PHPUnit events: `src/PhpUnit/Event/`
- Rector rules: `src/Rector/`
- Composer plugin: `src/Composer/`

**Architecture Direction:**

- Avoid introducing new dependencies on `composer/composer` outside the
  existing Composer plugin integration and legacy surfaces already awaiting
  decoupling.
- Prefer DevTools-owned interfaces for generic runtime concerns such as
  environment variables, process execution, filesystem access, and console
  presentation instead of reaching for Composer utility classes.
- When an existing Composer utility is convenient, first check whether a small
  local abstraction would support the ongoing Composer decoupling work with
  minimal code.
- During the Composer-to-Symfony command migration, preserve the global
  execution affordances already relied on by nested tools: `--ansi`/`--no-ansi`,
  cache and `--cache-dir` handling, and working-directory behavior.
- Keep color behavior explicit in command wrappers for tools with known flags
  instead of probing binaries dynamically; for example, Symfony/Composer-style
  tools can receive `--ansi`, while PHPUnit uses `--colors=always`.
- Keep child-process environment policy centralized in `ProcessQueue`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [php-fast-forward/dev-tools](https://github.com/php-fast-forward/dev-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

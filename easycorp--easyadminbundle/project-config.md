---
trigger: always_on
description: Welcome, AI assistant. Please follow these guidelines when contributing to this repository.
---

# AI Contribution Guidelines

Welcome, AI assistant. Please follow these guidelines when contributing to this repository.

## Project Overview

EasyAdminBundle is a third-party Symfony bundle for creating admin backends. It provides CRUD controllers, dashboard management, and extensive field/filter configuration.

**Requirements:** PHP 8.2+, Symfony 6.4/7.x/8.x, Doctrine ORM 2.20+ or 3.6+

## General Rules

- Language: American English for code, comments, commits, branches
- Code quotes: wrap strings with single quotes in PHP, CSS, JavaScript
- Text quotes: straight quotes only (`'` and `"`, no typographic)
- Security: prevent XSS, CSRF, injections, auth bypass, open redirects

### Do Not Edit
- `vendor/` - managed by Composer
- `node_modules/` - managed by Yarn
- `var/` - Symfony cache/logs
- `public/bundles/` - generated assets
- `composer.lock`, `yarn.lock` - update only via package manager commands

## Architecture

### Key Patterns
- **Factory**: ActionFactory, EntityFactory, FieldFactory, FilterFactory, FormFactory
- **Configurator Chain**: FieldConfiguratorInterface, FilterConfiguratorInterface
- **Context Facade**: AdminContext wraps Request/Crud/Dashboard/I18n contexts
- **DTO Layer**: Type-safe data transfer (ActionDto, EntityDto, FieldDto, etc.)
- **Event Subscriber**: AdminRouterSubscriber, CrudAutocompleteSubscriber
- **Registry**: AdminControllerRegistry, TemplateRegistry
- **Provider**: AdminContextProvider, FieldProvider
- **Typed Collections**: FieldCollection, ActionCollection, FilterCollection
- **Argument Resolver**: AdminContextResolver, BatchActionDtoResolver

Don't modify the public API in `Contracts/` lightly — it's a stable, versioned interface that third-party code depends on.

### Design Principles
- Never surface internal complexity to end users. Keep the public API small, simple, and consistent; push complexity down into internal classes (factories, configurators, DTOs), not onto the people configuring a backend.
- Public APIs are stable contracts: prefer additive, backward-compatible changes and don't leak internal types or implementation details through them.

### Backward Compatibility
Follow [Symfony's BC policy](https://symfony.com/bc): never break public API — deprecate first, keep the old path working, remove only in the next major. Emit deprecations with `@trigger_deprecation('easycorp/easyadmin-bundle', '<version>', '<message>')`, naming the replacement and removal version.

### Configuration Model
User-facing config uses fluent **builders** (`Config/`, `Field/`): static `new()` + chainable setters returning `self`. Each builder converts via `getAsDto()` into an immutable **DTO** (`Dto/`) for internal use. New configurable concepts follow this split — fluent builder for users, DTO internally; never expose a DTO as the config surface.

### Main Namespace
`EasyCorp\Bundle\EasyAdminBundle\`

## Commands

The `Makefile` is the source of truth for build/lint/test commands; its targets match what CI runs. Prefer them over hand-typed invocations. Run `make help` to list every target.

### Setup
```bash
make build          # composer update + yarn install (first-time setup)
composer install    # PHP dependencies only
yarn install        # JS dependencies only
```

### Before Creating a PR
Run the full check suite (linters + tests, identical to CI):
```bash
make checks-before-pr
```

### Pre-Commit Checklist
Run the relevant target(s) for what you changed:

| Changed          | Run                                                               |
|------------------|-------------------------------------------------------------------|
| PHP code         | `make linter-phpstan`, `make linter-cs-fixer`, `make tests`       |
| JS / CSS         | `yarn ci`, then `make build-assets`                               |
| Twig templates   | `make linter-twig`                                                |
| Documentation    | `make linter-docs`                                                |
| Translations     | keep all locales consistent; use English as placeholder if unsure |

Run targeted tests with `make tests ARGS=...` (keeps the deprecation baseline and cache reset; full guide in `tests/AGENTS.md`):
```bash
make tests                            # whole suite
make tests ARGS="tests/Unit/Field/"   # one directory
make tests ARGS="--filter=testFoo"    # one test
```
A bare `./vendor/bin/simple-phpunit` run skips the deprecation baseline (`tests/baseline-ignore.txt`) and the cache reset.

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

php-cs-fixer (`@Symfony` + `@Symfony:risky`) auto-formats most style — trailing commas, braces, strict comparisons, Yoda conditions, blank lines, quotes, etc. Run `make linter-cs-fixer` and let it handle formatting. The rules below are the conventions it does **not** enforce.

### Conventions
- PHP 8.2+ syntax with constructor property promotion

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EasyCorp/EasyAdminBundle](https://github.com/EasyCorp/EasyAdminBundle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

---
trigger: always_on
description: A Composer package that provides a CLI tool to bootstrap new Laravel projects with opinionated conventions, packages, and project structure.
---

# Kickoff - Laravel Project Bootstrap CLI

A Composer package that provides a CLI tool to bootstrap new Laravel projects with opinionated conventions, packages, and project structure.

## Architecture Overview

**Package Type**: Symfony Console application distributed as global Composer package
**Purpose**: Automate setup of new Laravel projects with pre-configured packages, stubs, and workflows
**Entry Point**: `bin/kickoff` executable
**Main Command**: `StartCommand` in `src/StartCommand.php`

### Key Components

1. **CLI Binary** (`bin/kickoff`): Symfony Console application entry point
2. **StartCommand** (`src/StartCommand.php`): Orchestrates the entire setup process
3. **Helper Functions** (`support/helpers.php`): Utilities for file operations and command execution
4. **Stubs Directory** (`stubs/`): Complete template structure copied to target Laravel projects

### How It Works

```bash
kickoff start <owner> <project-name> [<project-path>]
```

The command executes these steps sequentially:

1. Validates target is a Laravel project (checks for `artisan` file)
2. Copies entire `stubs/` directory to project
3. Modifies `composer.json` with custom scripts and autoload rules
4. Updates placeholders (`${PROJECT_NAME}`, `${OWNER}`) in files
5. Installs required Composer packages (Spatie, Laravel Telescope, etc.)
6. Publishes vendor configs and migrations
7. Installs NPM dependencies (tippy.js)
8. Runs project setup scripts (`bin/install`, migrations, asset build)

## Development Conventions

### Code Style

- **PHP Version**: 8.2+ (package supports Laravel 10-12)
- **Formatting**: Use Laravel Pint (`composer format`)
- **Static Analysis**: PHPStan Level 0 (`composer analyse`)
- **Testing**: PHPUnit (NOT Pest - this is the package itself, not generated projects)

### File Organization

- `src/`: Command classes (currently only `StartCommand`)
- `support/`: Helper functions auto-loaded via Composer
- `stubs/`: Complete project template structure
- `tests/`: PHPUnit tests for command functionality
- `bin/`: Executable entry point

### Helper Functions (`support/helpers.php`)

All helpers are designed for CLI operations:

```php
step(string $message, callable $callback, OutputInterface $output, bool $verbose)
// Wraps operations with loading indicators (⏳ → ✅/❌)

runCommand(string $cmd, bool $verbose)
// Executes shell commands with optional output suppression

installPackages(array $require, array $requireDev, string $path, bool $verbose)
// Installs Composer dependencies

copyRecursively(string $src, string $dst, bool $verbose, ?OutputInterface $output)
// Copies directory trees with optional verbose logging

ensureDir(string $path, int $mode = 0755)
// Creates directory if not exists

putFile(string $path, string $content)
// Writes content to file
```

### Placeholder Replacement System

Two placeholders are used throughout stubs:

- `${PROJECT_NAME}`: Replaced with project name argument
- `${OWNER}`: Replaced with owner argument

These appear in:

- `stubs/README.md`
- `stubs/.env.example`
- All `stubs/bin/*` scripts

**Implementation**: `updatePlaceholder()` method in `StartCommand`

## Testing Strategy

### Test File: `tests/StartCommandTest.php`

Uses PHPUnit with mocking to test:

- Command configuration (arguments: owner, name, path)
- Getter methods for project properties
- Mock execution to verify workflow

**Run Tests:**

```bash
composer test          # Runs PHPUnit
composer test-coverage # With coverage report
```

### Testing Approach

Since this is a file-system heavy operation, tests focus on:

1. Command configuration validation
2. Method accessibility and return values
3. Mock-based workflow verification

**Note**: Integration tests that actually create projects are not included (would require Laravel installation)

## Stubs Architecture

The `stubs/` directory contains a **complete Laravel project structure** that gets copied to target projects.

### Critical Stub Files

**Configuration Files:**

- `stubs/rector.php`: PHP 8.3, Laravel 11 level set
- `stubs/pint.json`: Relaxed PHPDoc rules
- `stubs/phpunit.xml`: Test environment settings
- `stubs/tailwind.config.js`: TailwindCSS v4 configuration
- `stubs/docker-compose.yml`: MinIO, Elasticsearch, Redis services

**Project Scripts** (`stubs/bin/`):

- `install`: Creates database, updates .env, runs migrations
- `deploy`: Git-based deployment script
- `backup-app`, `backup-media`: Backup utilities
- `build-fe-assets`, `reinstall-npm`, `update-dependencies`: Build tools

**Custom Stubs** (`stubs/stubs/`):

- `model.stub`: Extends `App\Models\Base` (not Eloquent Model)
- `migration.create.stub`: Dual-key pattern (auto-increment `id` + `uuid` column)
- `pest.stub`: Pest syntax for tests
- `policy.stub`: Standard policy methods

**Helper Functions** (`stubs/support/`):

- Organized by domain: `user.php`, `flash.php`, `media.php`, `menu.php`, etc.
- `helpers.php` uses `require_all_in(__DIR__.'/*.php')` pattern
- All wrapped in `function_exists()` checks

**Documentation Placeholders:**

- `CHANGELOG.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`
- `docs/` with deployment, development, and standards subdirectories

### Stubs vs Package Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cleaniquecoders/kickoff](https://github.com/cleaniquecoders/kickoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->

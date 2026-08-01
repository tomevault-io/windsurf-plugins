---
trigger: always_on
description: Project: Laravel-Lang / lang
---

# General information

Project: Laravel-Lang / lang

This document captures project-specific knowledge to help advanced contributors set up, test, and develop efficiently. It focuses on tools, commands, and conventions that are unique to this repository.

## Build / Configuration

- Runtime requirements
  - PHP: ^8.2. The test suite uses PHPUnit ^11|^12.
  - PHP extensions: ext-json.
  - Composer 2.x.
  - Node.js is optional and only needed for Biome (JSON/JS formatting) and npm-related codestyle tasks.

- Installation
  - Install PHP and Composer as usual.
  - From the repository root:
    - composer install
  - The repository ships a composer.lock; use --prefer-dist for faster installs in CI/CD if desired.

- Autoloading
  - Production code: PSR-4 autoload for namespace LaravelLang\Lang\ -> src/ (composer.json → autoload).
  - Tests: PSR-4 dev autoload for namespace Tests\ -> tests/ (composer.json → autoload-dev).

- Laravel integration
  - The package provides a ServiceProvider (LaravelLang\Lang\ServiceProvider) declared under extra.laravel.providers to allow auto-discovery in Laravel apps.

- Scripts / toolchain
  - composer scripts
    - format → runs vendor/bin/lang sync and then @style (Pint). Use when synchronizing locales from sources.
    - style → vendor/bin/pint --parallel (applies PHP code style rules defined in pint.json).
    - post-update-cmd → runs codestyler tasks and composer normalize on dependency updates. You generally don’t call this directly.
  - Pint (pint.json)
    - Preset: laravel with several project-specific rules, including declare_strict_types, fully_qualified_strict_types, php_unit_method_casing=camel_case, binary_operator_spaces alignment minimal, and multiple PHP migration rule sets (@PHP8xMigration).
    - Excludes tests/Fixtures from styling.
  - Biome (biome.json)
    - Used for JSON/JS formatting and linting outside of vendor/node_modules/etc.
    - VCS integration disabled; it won’t honor .gitignore by default (useIgnoreFile=false). The config explicitly excludes node_modules, vendor, and several root files.

## Testing

- Framework / configuration
  - PHPUnit is configured via phpunit.xml at the project root.
  - Bootstrap: vendor/autoload.php
  - Env: APP_KEY is set in phpunit.xml; you generally don’t need to export env vars for tests.
  - Source includes: ./src is included for code coverage/resolution.
  - The suite depends on laravel-lang/status-generator for its base TestCase; see tests/PluginTest.php extending LaravelLang\StatusGeneratorTests\TestCase.

- Running tests
  - Full suite:
    - vendor\bin\phpunit -c phpunit.xml --testdox
  - Filter by class or test name (useful for quick feedback):
    - vendor\bin\phpunit -c phpunit.xml --filter PluginTest --testdox
  - On Windows PowerShell, mind backslashes in paths; on *nix, use forward slashes.

- Adding tests
  - Namespaces: place tests under namespace Tests; files in tests/ are autoloaded via composer.json autoload-dev.
  - Base test class options:
    1) For repository-internal, framework-agnostic checks, extend PHPUnit\Framework\TestCase directly.
    2) For integration with the project’s locale/status infrastructure, extend LaravelLang\StatusGeneratorTests\TestCase (see tests/PluginTest.php) to leverage the helpers and fixtures provided by the status-generator package.
  - Test naming / conventions:
    - Use camelCase for test methods (enforced via pint rule php_unit_method_casing=camel_case).
    - Declare strict types in test files (declare(strict_types=1);) per coding style.

- Example: Creating and running a simple test (validated)
  - Minimal example (PHPUnit-only):
    - File: tests/DemoExampleTest.php
    - Content:
      <?php
      declare(strict_types=1);
      namespace Tests;
      use PHPUnit\Framework\TestCase;
      final class DemoExampleTest extends TestCase
      {
          public function test_it_runs_a_trivial_assertion(): void
          {
              $this->assertSame(2, 1 + 1);
          }
      }
  - Run just this test:
    - vendor\bin\phpunit -c phpunit.xml --filter DemoExampleTest --testdox
  - Note: This example was created and executed during documentation authoring and then removed to keep the repository clean.

## Additional Development Information

- Repository purpose
  - This package provides and maintains language resources for Laravel and related first-party packages. Locales live under locales/. The source/ directory holds inputs used to generate/update locales; automation is handled by laravel-lang/publisher and related tools.

- Directory overview
  - src/ — Package code, including the ServiceProvider and any helpers used at runtime.
  - source/ — Data sources used to generate translation files (don’t edit generated outputs directly without understanding the pipeline).
  - locales/ — Generated/maintained translation artifacts per locale (e.g., php-inline.json, json.json, _excludes.json). Some files may be machine-maintained; use sync tooling to update.
  - tests/ — Test suite; currently includes PluginTest which relies on status-generator’s BaseTestCase.
  - docs/ — Additional project/user documentation.

- Locale synchronization / formatting
  - To regenerate/synchronize locale files from sources:
    - composer run format

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Laravel-Lang/lang](https://github.com/Laravel-Lang/lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->

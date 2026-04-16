---
trigger: always_on
description: This is the development repository for mu-plugins (must-use plugins) that power the WordPress VIP platform. It's a large WordPress plugin collection (~60+ top-level directories) written primarily in PHP, supporting 8.1, 8.2, 8.3, and 8.4.
---

# WordPress VIP MU-Plugins - Coding Agent Instructions

## Repository Overview

This is the development repository for mu-plugins (must-use plugins) that power the WordPress VIP platform. It's a large WordPress plugin collection (~60+ top-level directories) written primarily in PHP, supporting 8.1, 8.2, 8.3, and 8.4.

**Key Characteristics:**
- Type: WordPress mu-plugins collection
- Primary Language: PHP (with JavaScript for WordPress admin features and E2E tests)
- Package Manager: Composer for PHP dependencies, npm for Node.js tooling
- Git Submodules: 8 submodules (jetpack, elasticpress, wp-parsely, akismet, etc.)
- Testing: PHPUnit (Docker-based), E2E tests (Playwright), Core WordPress tests
- CI/CD: GitHub Actions workflows (no CircleCI despite references in README)

## Critical Setup Steps

### ALWAYS Run These Commands First

Dependencies MUST be installed in this exact order before any other operations:

```bash
git submodule update --init --recursive
composer install --no-interaction
npm install
```

**Important:** Composer install can take 5-10 minutes. Use `--no-interaction` flag to prevent hangs. Never skip submodule initialization - many directories are submodules and tests will fail without them.

## Build, Test, and Validation Commands

### PHP Linting

```bash
npm run phplint
```
- Runs in ~5 seconds
- Uses npx phplint on all PHP files except vendor/, node_modules/, jetpack*/, wp-parsely*/
- Always run before committing PHP changes

### PHP Code Standards (PHPCS)

```bash
npm run phpcs
```
- Uses WordPress VIP coding standards (see `phpcs.xml.dist`)
- Scans incrementally via lint-staged on pre-commit (see `.husky/pre-commit` and `.lintstagedrc`)
- Full scan takes 30-60 seconds
- Configuration: PHP 8.1+ compatibility, WordPress 6.2+ support
- Excludes: vendor/, submodules (advanced-post-cache, cron-control, http-concat, jetpack, etc.), __tests__/, bin/

To fix auto-fixable issues:
```bash
npm run phpcs:fix
```

### PHPUnit Tests

**Primary Method (Docker-based):**
```bash
./bin/test.sh
```
- Self-contained Docker environment (downloads MySQL 8 and test runner images on first run)
- Takes 2-10 minutes for full test suite
- Uses `phpunit.xml` configuration
- Supports filtering: `./bin/test.sh --filter test_name_here`

**Options:**
- `--wp VERSION`: WordPress version (default: latest)
- `--multisite yes/no`: Enable multisite mode
- `--php VERSION`: PHP version
- `--filter PATTERN`: Run specific tests

The script creates a Docker network and MySQL container automatically. It will clean up on exit.

**Common Test Failures:**
- If Docker images aren't present, first run will download them (can take 5-10 minutes)
- Ensure Docker daemon is running
- Database connection failures: script waits up to 60 seconds for MySQL to start

### End-to-End Tests

E2E tests are in `__tests__/e2e/` using Playwright and TypeScript:

```bash
cd __tests__/e2e
npm ci
npx playwright install chromium
npm test  # Sets up environment, runs tests, tears down
```

**Note:** E2E tests require VIP CLI (`npm install -g @automattic/vip`). Tests automatically manage test environment lifecycle.

### Run All Quality Checks

```bash
npm run lint  # Runs both phplint and phpcs
```

## GitHub Actions CI Pipelines

All PRs to `develop` branch trigger these workflows:

1. **CI Workflow** (`.github/workflows/ci.yml`):
   - Runs PHPUnit tests across matrix of WP versions (6.4.x-nightly), PHP versions (8.1-8.4), multisite yes/no, Jetpack yes/no
   - Uses MySQL 8 service container
   - Takes about 10 minutes for full matrix
   - Generates code coverage for latest WP + PHP 8.1

2. **Lint Workflow** (`.github/workflows/lint.yml`):
   - Runs `npm run lint` (phplint + phpcs)
   - Also lints Search Dev Tools separately
   - Takes 1-2 minutes

3. **E2E Workflow** (`.github/workflows/e2e.yml`):
   - Runs Playwright E2E tests
   - Includes ESLint check for E2E test code
   - Takes 5-10 minutes

4. **Core Tests Workflow** (`.github/workflows/core-tests.yml`):
   - Runs WordPress core PHPUnit tests with mu-plugins loaded
   - Complex setup, takes 15-20 minutes
   - Many core tests are ignored (see workflow for filters)

## Repository Structure

### Root-Level Files
- `000-vip-init.php`: VIP platform initialization (loaded first)
- `001-core.php`, `001-cron.php`: Core mu-plugin loaders
- `z-client-mu-plugins.php`: Loads client-specific mu-plugins (loaded last)
- `composer.json.tpl`: Template for generated composer.json
- `phpcs.xml.dist`: PHPCS configuration
- `phpunit.xml`, `phpunit-multisite.xml`: PHPUnit configurations

### Key Directories

**Core Functionality:**
- `search/`: Enterprise Search integration (ElasticPress, ES-WP-Query, search-dev-tools)
- `cache/`: Caching layer implementations
- `security/`: Security features
- `performance/`: Performance optimizations
- `lib/`: Shared library code
- `vip-helpers/`: VIP platform helper functions

**WordPress Integration:**
- `jetpack/`: Jetpack (submodule)
- `wp-parsely/`: Parse.ly plugin (submodule)
- `akismet/`: Akismet (submodule, not in PHPCS scope)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Automattic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

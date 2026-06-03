---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lottie Lite is a WordPress plugin that extends core image blocks (Image, Cover, Media & Text) with support for Lottie animations. The plugin allows overlaying or replacing images with Lottie animations, with responsive breakpoints and interactive triggers.

## Build Commands

```bash
# Build production assets
npm run build

# Development watch mode
npm run start

# Linting
npm run lint:js        # Check JavaScript
npm run lint:css       # Check CSS
npm run lint:js:fix    # Fix JavaScript issues
npm run lint:css:fix   # Fix CSS issues

# Format code
npm run format
```

The build process uses `@wordpress/scripts` to compile JavaScript and CSS from `src/*.js` into the `build/` directory. Three entry points are compiled:
- `src/editor.js` → `build/editor.js` (Block editor interface)
- `src/lottie.js` → `build/lottie.js` (Frontend runtime)
- `src/media-view.js` → `build/media-view.js` (WordPress media library integration)

## Testing

The plugin uses Playwright for end-to-end testing with WordPress Playground.

```bash
# Start WordPress Playground server (local development)
npm run playground:start

# Run e2e tests (in another terminal)
npm run test:e2e

# Run tests in debug mode
npm run test:e2e:debug

# Run tests in watch mode
npm run test:e2e:watch
```

WordPress Playground runs on `http://localhost:9400` with:
- WordPress 6.8
- PHP 8.3
- Auto-login as admin/password
- Debug mode enabled

The environment is configured via `blueprint.json` which defines:
- WordPress and PHP versions
- Auto-login credentials
- Theme installation (twentytwentyfour)
- Plugin activation
- Debug constants

Test files are in `tests/e2e/` and use Playwright Test with `@wordpress/e2e-test-utils-playwright`.

### Blueprint Configuration

The `blueprint.json` file defines the WordPress Playground environment:
- `preferredVersions`: WordPress and PHP versions
- `steps`: Array of setup actions (login, install plugins/themes, set config)
- `landingPage`: Default page to load

Modify the blueprint to change the test environment configuration.

## CI/CD Pipeline

GitHub Actions workflows automate testing, linting, and releases:

### Lint (`.github/workflows/lint.yml`)
- Triggers on push/PR to main, master, or develop branches
- Runs JavaScript and CSS linters
- Ensures code quality and style consistency
- Fails the build if linting issues are found

### Playwright Tests (`.github/workflows/playwright-tests.yml`)
- Triggers on push/PR to main, master, or develop branches
- Builds the plugin and runs Playwright e2e tests
- Uses WordPress Playground CLI on port 9400
- Tests run in a matrix across multiple versions:
  - PHP: 8.3, 8.4
  - WordPress: 6.7, 6.8, latest
- Configured via `blueprint.json` with version overrides via CLI flags (`--php`, `--wp`)
- Uploads test results and failure artifacts (named per matrix combination)
- Posts test summaries as PR comments

### PR Preview (`.github/workflows/pr-playground-preview.yml`)
- Triggers when PRs are opened or updated
- Replaces `__VERSION__` with "Pull Request #XXX - PR Title" for easy identification
- Builds the plugin with all assets
- Pushes built files to a branch named `pr-{number}-built`
- Generates a Playground blueprint that installs the plugin from GitHub
- Adds an interactive "Try it in Playground" button to PR descriptions
- Allows reviewers to test changes directly in their browser
- No local setup required
- Uses WordPress 6.8 with PHP 8.3

The workflow uses the same approach as the build-and-release workflow, pushing built assets to dedicated PR branches. The Playground blueprint installs the plugin directly from the GitHub branch ZIP. The plugin version in WordPress will show the PR number and title, making it clear which preview is being tested.

### PR Cleanup (`.github/workflows/pr-cleanup.yml`)
- Triggers when PRs are closed (merged or not)
- Automatically deletes the `pr-{number}-built` branch
- Keeps the repository clean by removing temporary preview branches
- No manual intervention required

### Build & Release (`.github/workflows/build-and-release.yml`)
- Triggers on push to main branch
- Builds production assets via `npm run build`
- Merges compiled assets to `release` branch
- The `release` branch contains only production-ready files (excludes src, tests, node_modules)

Files excluded from release builds are defined in `.gitattributes`.

### Version and Release (`.github/workflows/release.yml`)
- Triggers when a GitHub release is created
- Extracts version from the release tag (e.g., `v1.3.0` → `1.3.0`)
- Replaces `__VERSION__` placeholder in `lottie-lite.php` with actual version
- Commits the versioned file back to the tag
- Creates a clean ZIP archive excluding development files
- Uploads the ZIP as a release asset

## Release Process

The plugin uses `__VERSION__` as a placeholder in `lottie-lite.php` that gets replaced during releases:

```php
/**
 * Version: __VERSION__
 */
```

To create a new release:

1. Ensure all changes are merged to `main` branch
2. Create a new GitHub release with a version tag (e.g., `v1.3.0`)
3. The release workflow automatically:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [humanmade/lottie-lite](https://github.com/humanmade/lottie-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->

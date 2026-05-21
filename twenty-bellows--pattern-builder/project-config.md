---
trigger: always_on
description: This file provides guidance to Claude Code and other AI coding agents when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code and other AI coding agents when working with code in this repository.

## Project Overview

**Pattern Builder** is a WordPress plugin developed by [Twenty Bellows](https://twentybellows.com). It allows WordPress users to create, edit, organize, and manage block patterns directly in the admin interface — unifying theme patterns (PHP files) and user-created patterns (custom post type) in a single, intuitive UI with visual editing, code editing, live preview, and export capabilities.

- **Version:** 1.0.4
- **Repository:** https://github.com/twenty-bellows/pattern-builder
- **Issue Tracker:** GitHub Issues — https://github.com/twenty-bellows/pattern-builder/issues
- **Plugin URI:** https://www.twentybellows.com/pattern-builder/
- **License:** GPL-2.0-or-later
- **WordPress Requires:** 6.6+
- **PHP Requires:** 7.2+

## Development Environment

## Architecture (Key Design Decisions)

A full architectural analysis is in [`docs/architecture.md`](docs/architecture.md). Key decisions to understand before working in this codebase:

**The core problem:** WordPress's block editor can only edit things with a post ID. File-based theme patterns (`.php` files in `/patterns/`) have no post ID. The plugin solves this with a **DB mirror + REST hijacking** strategy.

**DB Mirror (`tbell_pattern_block` CPT):** Each theme pattern file gets a corresponding `tbell_pattern_block` post that gives it a database identity. This post is the source of the post ID the editor needs. The file remains the source of truth; the DB record is kept in sync.

**REST Hijacking:** The plugin intercepts `/wp/v2/blocks` requests at three filter points:
- `rest_request_after_callbacks` (GET) — injects theme pattern posts into the blocks response so the editor sees them alongside user patterns
- `rest_pre_dispatch` (PUT/DELETE) — intercepts saves and deletes before the real handler runs, writing changes to the PHP file on disk instead of (or in addition to) the DB

**Pattern Registration (on `init`):** On every page load, the plugin globs the theme's `/patterns/` directory and upserts DB records for any new or changed patterns. This is a known performance issue (TWE-369) — no caching yet.

**Editor Integration:** Two things happen in the editor:
- `syncedPatternFilter` intercepts `core/pattern` blocks to enable editing synced theme patterns in context
- `PatternPanelAdditionsPlugin` adds sidebar panels (Source, Sync Status, Associations) when editing a `wp_block` post

**Admin Page:** Plain PHP (Appearance → Pattern Builder). Links to documentation. No JS.

**Companion Plugin:** [`synced-patterns-for-themes`](https://github.com/Twenty-Bellows/synced-patterns-for-themes) is a read-only subset of this plugin for production use. It uses the same REST hijacking approach but blocks edits. It self-deactivates when Pattern Builder is active.

---

## Development Environment

### Prerequisites
- Node.js (v18+ recommended)
- PHP 7.2+ with Composer
- Docker (for `wp-env` local WordPress environment and PHP integration tests)

### Environment Notes
- Docker is available (host socket shared). All `wp-env` commands work.
- `wp-env` binary is at `node_modules/.bin/wp-env` — run via npm scripts from this directory.
- First `npm run start` will pull WordPress Docker images (~1-2 min).

### Known Pre-Existing Issues
- Several PHP lint violations exist in the codebase (Yoda conditions, inline comment formatting). These are pre-existing and not regressions. Fix them if you touch the file; don't feel obligated to fix unrelated files.

## Development Commands

### Build Commands
- `npm run build` - Production build with minification
- `npm run watch` - Development build with hot reload
- `npm run format` - Format JavaScript code
- `npm run lint:js` - Lint JavaScript files
- `npm run lint:css` - Lint CSS/SCSS files
- `composer format` - Format PHP code using WordPress coding standards
- `composer lint` - Lint PHP code

### Testing Commands
- `npm run test:unit` - Run JavaScript unit tests (no Docker required)
- `npm run test:unit:watch` - Run JavaScript tests in watch mode
- `npm run test:php` - Run PHP unit tests in wp-env environment (**requires Docker**)
- `npm run test:php:watch` - Run PHP tests in watch mode (**requires Docker**)
- `composer test` - Run PHP tests directly via PHPUnit (requires WP test bootstrap)

### Development Environment (Docker required)
- `npm run start` - Start wp-env with xdebug enabled
- `npm run stop` - Stop wp-env
- `npm run clean` - Clean wp-env
- `npm run plugin-test-env` - Start WP Playground for testing
- `npm run plugin-test` - Full build, zip, and test workflow

## Architecture Overview

### Plugin Structure
The plugin follows a component-based OOP architecture with clear separation of concerns:

1. **Main Entry Point**: `pattern-builder.php` initializes the plugin
2. **Core Class**: `Pattern_Builder` (singleton in `includes/class-pattern-builder.php`) bootstraps all plugin components
3. **Component Classes** (`includes/`):
   - `Pattern_Builder_API` - REST API endpoints under `/pattern-builder/v1/`
   - `Pattern_Builder_Admin` - Admin UI under Appearance → Pattern Builder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Twenty-Bellows/pattern-builder](https://github.com/Twenty-Bellows/pattern-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->

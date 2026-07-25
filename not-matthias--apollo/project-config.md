---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Apollo** is a modern, minimalistic blog theme for Zola (a static site generator). The repository includes the theme itself, demonstration content, and comprehensive UI testing infrastructure using Playwright.

- Theme Documentation: See `README.md` for installation and configuration options
- Configuration: `config.toml` contains site-wide settings and theme options
- Test Documentation: Detailed testing information in `tests/README.md`

## Development Environment Setup

The project uses a **Nix flake** for reproducible development. With direnv enabled, the environment is automatically loaded.

```bash
# Manual entry (if direnv not configured)
nix-shell

# Or use direnv
direnv allow
```

Dependencies provided via flake:
- **zola**: Static site generator
- **bun**: JavaScript runtime for Playwright tests and npm scripts
- **pre-commit**: Git hooks enforcement
- **just**: Task runner (alternative to Make)
- **treefmt**: Multi-language code formatter
- **prettier**, **alejandra**, **djlint**: Specific formatters
- **minify**: Asset minification tool

## Common Commands

### Build & Serve

```bash
# Build static site to ./dist
just build

# Serve locally with live reload (development)
bun run serve
# Server runs at http://127.0.0.1:1111

# Build and run Lighthouse performance audit
just lighthouse

# View Lighthouse results
just lighthouse-open

# Clean generated files
just clean
```

### Testing

All tests use **Playwright** and require the dev environment to be active. The Zola server starts automatically during testing.

**⚠️ IMPORTANT: Always run tests in Docker** to match the CI environment exactly and avoid browser compatibility issues on NixOS.

```bash
# Install dependencies first
bun install

# Run all tests in Docker (RECOMMENDED)
bun run test:docker-compose

# Run all tests locally (may have browser issues on NixOS)
bun test

# Run tests with interactive UI
bun run test:ui

# Run tests with browser visible (headed mode)
bun run test:headed

# Debug specific test (step through)
bun run test:debug

# Update visual regression baselines (after intentional changes)
# IMPORTANT: Always use Docker to ensure consistent screenshots
docker-compose -f docker-compose.test.yml run --rm playwright-tests npm run test:update-snapshots

# Run specific test suite
bunx playwright test tests/theme/         # Theme tests only
bunx playwright test tests/navigation/    # Navigation tests only
bunx playwright test tests/visual/        # Visual regression tests only
bunx playwright test tests/content/       # Content rendering tests only
```

Test reports are generated in `playwright-report/` and can be viewed with:
```bash
bunx playwright show-report
```

### Code Formatting

The project uses **treefmt** with language-specific formatters:

```bash
# Format all tracked files
treefmt

# Format specific file types
prettier --write "sass/**/*.scss"  # SCSS
djlint --reformat templates/       # HTML
alejandra --check *.nix            # Nix
```

Pre-commit hooks automatically run formatting on git commit (see `.pre-commit-config.yaml`).

## Project Architecture

### Directory Structure

```
apollo/
├── content/              # Site content (blog posts, projects, pages)
│   ├── posts/           # Blog post markdown files
│   ├── projects/        # Projects page content
│   ├── talks/          # Talks page content
│   └── _index.md       # Homepage
├── sass/                # Theme styling (SCSS)
│   ├── main.scss       # Main stylesheet entry point
│   ├── parts/          # Component styles
│   ├── theme/          # Theme-specific styles (light/dark)
│   └── fonts.scss      # Font definitions
├── static/             # Static assets (favicons, images, etc.)
├── tests/              # Playwright UI tests
│   ├── theme/          # Theme switching tests
│   ├── navigation/     # Menu and TOC tests
│   ├── content/        # Content rendering tests
│   ├── visual/         # Visual regression tests
│   ├── helpers.ts      # Common test utilities
│   └── screenshots/    # Baseline screenshots for visual tests
├── config.toml         # Zola configuration (site settings, theme options)
├── flake.nix          # Nix development environment
├── playwright.config.ts # Playwright test configuration
└── Justfile           # Task automation

```

### Key Configuration Files

**config.toml**: Site configuration and theme settings
- `theme = "toggle"`: Theme switching behavior (light/dark/auto/toggle)
- `build_search_index = true`: Enables site search
- `compile_sass = true`: Builds CSS from SCSS
- `taxonomies = [{ name = "tags" }]`: Tag system for posts
- `[extra]` section: Theme-specific features (MathJax, Mermaid, analytics, etc.)

**playwright.config.ts**: Test configuration
- Runs against multiple browsers (Chromium, Firefox, WebKit)
- Tests mobile viewports (iPhone 12, Pixel 5)
- Visual regression threshold: 2% max pixel difference
- Base URL: `http://127.0.0.1:1111` (Zola dev server)
- Auto-starts Zola before running tests

### Testing Architecture

**Test Organization**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [not-matthias/apollo](https://github.com/not-matthias/apollo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

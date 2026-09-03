---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

X Search Pro is a Chrome extension (Manifest V3) that allows users to build a personal library of X/Twitter searches with smart auto-updating time ranges. Built with vanilla JavaScript, no frameworks or build tools required.

**Key Feature**: Sliding window searches - searches with `slidingWindow` property ('1d', '1w', '1m') that automatically update their date ranges when applied, ensuring searches always show recent content without manual date adjustments.

## Development Commands

```bash
# Code quality
npm run lint              # ESLint - check for code issues
npm run typecheck         # TypeScript type checking

# Testing
npm test                  # Unit tests only (fast, no X.com credentials needed)
npm run test:install      # Install Playwright browsers (first time only)
npm run test:unit         # Same as npm test
npm run test:e2e          # Full E2E tests (requires .env with X.com credentials)
npm run test:e2e:headed   # E2E with visible browser
npm run test:e2e:ui       # Playwright UI mode
npm run test:e2e:debug    # Debug mode with inspector
npm run test:workflows    # Workflow tests only
npm run test:report       # Open HTML test report

# Pre-push validation (runs automatically via Husky)
npm run validate:pre-push # Lint + typecheck + unit + e2e tests

# Build
npm run build:zip         # Create distribution zip for Chrome Web Store
```

## Architecture

### Core Components

**Extension Structure** (Manifest V3):
- `manifest.json` - Extension configuration with permissions for storage, activeTab, and x.com/twitter.com hosts
- `background/service-worker.js` - Background service worker that initializes default templates on install
- `popup/` - Extension popup UI with tabs for Search Builder, Saved Searches, Categories, and Settings
- `content/content.js` - Content script injected into x.com/twitter.com pages that manages sidebar and applies searches
- `lib/` - Shared libraries loaded by both popup and content scripts

**Shared Libraries** (`lib/`):
- `query-builder.js` - `QueryBuilder` class that builds X search queries from filter objects
- `storage.js` - `StorageManager` singleton for Chrome storage operations (uses `chrome.storage.sync`)
- `templates.js` - Default search templates initialized on first install

### Data Flow

1. **Creating a search**: User fills form in popup → `QueryBuilder` generates query string → `StorageManager.saveSearch()` saves to Chrome sync storage
2. **Applying a search**: User clicks search (popup or sidebar) → Query rebuilt (with fresh dates if `slidingWindow` set) → Message sent to content script → `applySearchToPage()` fills search box on x.com
3. **Sliding windows**: When search has `filters.slidingWindow` ('1d'/'1w'/'1m'), `QueryBuilder.calculateSlidingDates()` recalculates `since`/`until` dates dynamically when building query

### Storage Schema

Uses `chrome.storage.sync` with these keys:
- `savedSearches` - Array of search objects with `{id, name, query, filters, category, color, isCustomColor, slidingWindow, useCount}`
- `categories` - Array of category names
- `categoryColors` - Object mapping category names to hex colors
- `sidebarVisible` - Boolean for sidebar toggle state
- `sidebarCollapsed` - Boolean for sidebar collapsed state
- `templatesInitialized` - Boolean flag to prevent re-initializing default templates

### Query Building System

`QueryBuilder` class supports:
- Engagement filters: `min_faves`, `min_retweets`, `min_replies` (also max versions using negative syntax)
- Date filters: Fixed dates OR sliding windows that auto-update
- User filters: `from:`, `to:`, `@mention`, `filter:blue_verified`, `filter:follows`
- Content filters: `filter:media/images/videos/links/replies/retweets/quote`
- Language: `lang:XX`

**Critical sliding window logic** (`query-builder.js:163-193`):
- If `filters.slidingWindow` is set, `calculateSlidingDates()` computes fresh dates relative to today
- This ensures searches with sliding windows always show recent content when applied

### Sidebar System

The sidebar (`content/content.js:66-147`) is injected into x.com pages:
- Persists across page navigations via `MutationObserver` watching for SPA route changes
- Toggle button pinned to right edge of screen
- Sidebar panel slides in/out with CSS transforms
- Can be collapsed to icon-only view
- Syncs with storage changes in real-time via `chrome.storage.onChanged` listener

## Testing

### Test Structure
- `tests/unit/` - Fast unit tests for QueryBuilder, storage, templates (97 tests)
- `tests/e2e/workflows/` - End-to-end tests using test page (fast, no auth required)
- `tests/fixtures/` - Custom Playwright fixtures for extension context
  - `test-page.html` - Lightweight HTML page that mimics X.com structure for testing
  - `extension.ts` - Extension fixture that uses `manifest.test.json` for file:// access
- `tests/page-objects/` - Page object models (PopupPage, SidebarPage)
- `tests/helpers/` - Test helpers
  - `test-page-helpers.ts` - TestPageHelpers for test page navigation
  - `x-page-helpers.ts` - XPageHelpers for X.com integration tests (optional, for manual testing)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neonwatty/x-search-pro](https://github.com/neonwatty/x-search-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->

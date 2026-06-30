---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Accessibility Visualizer browser extension - a web accessibility visualizer that helps developers identify accessibility issues by highlighting and annotating elements on web pages. The project is built with WXT framework and React, supporting both Chrome/Chromium and Firefox browsers.

## Requirements for Claude Code

Claude Coude should follow these guidelines when working with this repository:

- **Language**: When prompted in Japanese, always respond in Japanese. Internal thinking can be in any language, but the final response must match the language of the user's request.
- Use the `pnpm` package manager for all commands. Do not use `npm` or `yarn`.
- This browser extensions is enabled by many users who is engaged not only in an accessibility related work but also many other works. So ensure code quality and stability. Avoid introducing changes without proper testing.
- If you are asked something that includes some braking changes, please ask the developer to ensure it is acceptable before implementing it.
- **Before completing your work**: Always run `pnpm lint-fix` to format code with Prettier and fix linting issues. This ensures consistent code formatting across the project.
- **After any code changes**: Run `pnpm lint-fix` at the end of every work session to maintain code quality and formatting consistency.

## Commands

### Development

```bash
# Install dependencies
pnpm install

# Develop browser extension for Chrome/Chromium
pnpm dev
# or: pnpm --filter=@a11y-visualizer/browser-extension dev

# Develop browser extension for Firefox
pnpm dev:firefox
# or: pnpm --filter=@a11y-visualizer/browser-extension dev:firefox

# Develop website
pnpm dev:website
# or: pnpm --filter=@a11y-visualizer/website dev

```

### Building

```bash
# Build for Chrome/Chromium
pnpm build
# or: pnpm --filter=@a11y-visualizer/browser-extension build

# Build for Firefox
pnpm build:firefox
# or: pnpm --filter=@a11y-visualizer/browser-extension build:firefox

# Build website
pnpm build:website
# or: pnpm --filter=@a11y-visualizer/website build

# Create distribution zip files
pnpm zip
pnpm zip:firefox
```

### Testing & Linting

```bash
# Run tests across all packages
pnpm test

# Run linting across monorepo and packages
pnpm lint

# Fix linting issues
pnpm lint-fix

# Type check (browser extension)
pnpm --filter=@a11y-visualizer/browser-extension compile

# Run specific browser extension tests
pnpm --filter=@a11y-visualizer/browser-extension test

# Run tests for a specific rule
pnpm test src/rules/rule-name/index.test.ts
```

## Architecture

### Monorepo Structure

- `apps/browser_extension/` - Main WXT-based browser extension
- `apps/website/` - Promotional website with GitHub Pages deployment
- `docs/` - User guides in multiple languages

### Browser Extension Architecture

**Content Script (`entrypoints/content/`)**

- Main extension logic injected into web pages
- `collectElements.ts` - Core element collection and analysis
- `getElementCategory.ts` - Categorizes elements (image, heading, control, etc.)
- `getElementPosition.ts` - Calculates element positioning for overlays

**Accessibility Rules System (`src/rules/`)**

- Each rule is a separate module (e.g., `accessible-name/`, `aria-hidden/`)
- Rules implement `RuleObject` interface with evaluation logic
- Centrally registered in `Rules.ts`
- Returns `RuleResult[]` with error/warning/content/state information
- Rule testing follows consistent patterns:
  - Use `vitest` with `describe` and `test` blocks
  - Always clean up DOM with `afterEach(() => { document.body.innerHTML = ""; })`
  - Test both enabled/disabled states and various element conditions
  - Include tests for edge cases and multiple attribute scenarios

**Element Categorization**
Elements are categorized into: image, heading, control, section, list, listItem, table, tableCell, group, page, general

**Settings & Presets (`src/settings/`)**

- Configurable rule options and visual presets
- Browser storage integration
- Tab-based preset system for different element types

**UI Components**

- React components with Tailwind CSS styling
- Shadow DOM isolation for overlay elements
- Internationalization support (en/ja/ko)
- Test pages (`apps/website/src/components/tests/`) demonstrate extension functionality:
  - Organized by element types (Form Controls, Images, Headings, etc.)
  - Include both positive examples (✅) and negative examples (❌/⚠️)
  - Each example should be self-contained and clearly documented
  - Follow consistent styling patterns with border colors for problem indicators

### Key Extension Points

- Content script injection via `injectRoot.tsx`
- Background script for extension lifecycle
- Options page for configuration
- Popup for quick toggles

## Branding Guidelines

- Keep the simplicity and clarity for the end-users.
  - Do not menthion the internal implementation details such as the number of rules.
- The extension name is "Accessibility Visualizer"
  - Do not change the spelling or capitalization
  - In other languages, Do not translate the name, use "Accessibility Visualizere as is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ymrl/a11y-visualizer](https://github.com/ymrl/a11y-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

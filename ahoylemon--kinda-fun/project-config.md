---
trigger: always_on
description: Kinda Fun is a Vue.js-based web gaming platform featuring 6+ multiplayer and single-player games built by Lemon. The platform includes games like Invalid (multiplayer trivia game), Comparatively Famous (celebrity valutation via Cameo), No More Billionaires (guillotine simulator), This Meeting Has Points (multiplayer card game meant for virtual meetings), Pretend World (celebrity impersonator guessing game), Megachurch Tycoon (single player business simulator) and Sisyphus Clicker (parody clicker
---

# Kinda Fun Gaming Platform

Kinda Fun is a Vue.js-based web gaming platform featuring 6+ multiplayer and single-player games built by Lemon. The platform includes games like Invalid (multiplayer trivia game), Comparatively Famous (celebrity valutation via Cameo), No More Billionaires (guillotine simulator), This Meeting Has Points (multiplayer card game meant for virtual meetings), Pretend World (celebrity impersonator guessing game), Megachurch Tycoon (single player business simulator) and Sisyphus Clicker (parody clicker game).

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Prerequisites and Setup

- **Node.js Version**: Node.js 22+ is required for all development and deployment (updated due to Vite 6 compatibility requirements).
- **Environment Setup**:
  - `npm install` -- installs all dependencies including Firebase functions. Takes ~45 seconds. NEVER CANCEL.
  - Functions and main project now both use Node.js 22+.

### Dependency Update Policy

- **Dependabot is intentionally disabled for this repository.**
- Reason: for this single-maintainer project, Dependabot generated high-volume PR noise and repeated non-essential update churn, including breakage and review overhead that outweighed value.
- Do not reintroduce `.github/dependabot.yml` unless the maintainer explicitly asks for it.
- Handle dependency updates manually and only when needed (especially for confirmed security or compatibility needs).

### Build and Development Commands

- **Development Server**: `bun run dev:client` -- starts Vite dev server on http://localhost:5173.
- **Production Build**: `bun run build -- --mode production` -- builds all games and static pages in prodution mode (to be deployed to Firebase).
- **Development Build**: `bun run build -- --mode development` -- builds all games and static pages in development mode (for local testing and development).
- **Preview Build**: `bun run preview` -- serves built application on http://localhost:4173. Takes ~2 seconds.

### Testing and Quality

- **Unit Tests**: `bun run test:unit` -- runs Vitest test suite. Use this to validate changes.
- **Linting**: `bun run lint` -- runs ESLint with auto-fix.
- **Code Formatting**: `bun run format` -- formats all source files with Prettier. Run before committing.

### Data Generation Commands

- **Billionaire Data**: `bun run guillotine:js` -- generates billionaire data from CSV. Takes ~2 seconds. Creates 2,781 entries.
- **Arrest Warrants**: `bun run guillotine:arrests` -- creates daily arrest warrant data.
- **Page Building**: `bun run build:pages` -- builds static HTML pages from templates.

## Code Organization

### File Size Guideline

- **When a file exceeds ~900 lines, consider splitting it into partial/component files.**
- This is not a hard limit — it's a prompt to evaluate whether splitting would improve readability or maintainability.
- For Vue games, typical splits are: main component, sub-components, data/config files, and separate SCSS partials.

## Copilot Working Style Preferences

When implementing features, Copilot should:

- Create a detailed todo list before starting work
- Provide status updates while working through the implementation
- Explain what each step accomplishes and why it's needed
- Break complex features into smaller, manageable pieces
- Test changes as they're implemented when possible

## Validation

### Manual Testing Requirements

**ALWAYS manually test after making changes to games or build system:**

1. **TypeScript Validation**:
   - Run `npx tsc --noEmit` to verify TypeScript compilation is clean

2. **Build Validation**:
   - Run `bun run build -- --mode production` and verify it completes without errors
   - Check that `dist/` folder contains all game HTML files (cameo.html, guillotine.html, invalid.html, etc.)

### Pre-Commit Requirements

**ALWAYS run before committing changes:**

- `bun run tsc` -- verifies TypeScript compilation is clean
- `bun run format` -- formats code consistently (but CHECK for SCSS function formatting issues)
- `bun run build -- --mode production` -- ensures production build works (NEVER CANCEL - takes ~10 seconds)

**CRITICAL: Prettier Formatting Warning**

- Prettier may incorrectly format SCSS functions like `lighten($color, $amount)` into multi-line named parameter syntax
- ALWAYS test `bun run build` after formatting to catch SCSS syntax errors
- Fix any SCSS function calls that Prettier breaks by converting them back to simple syntax

**CI will fail if:**

- Build fails (including SCSS compilation errors)
- Linting has new errors beyond current baseline
- Firebase deployment keys are missing (production only)

## Commit and Pull Request Guidelines

### Commit Message Format


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AhoyLemon/kinda.fun](https://github.com/AhoyLemon/kinda.fun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->

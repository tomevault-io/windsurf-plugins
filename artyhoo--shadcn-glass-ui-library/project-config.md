---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Quick Commands Cheatsheet

```bash
# Development
npm run dev                      # Vite dev server
npm run storybook               # Storybook on :6006

# Testing
npx vitest                      # All tests
npx vitest --ui                 # Vitest UI
npx vitest --project=storybook  # Component tests only
npx vitest --project=visual     # Visual regression only
npx vitest ComponentName        # Specific component
npm run test:visual:update      # ⚠️ macOS only for local preview
gh workflow run update-screenshots.yml  # ✅ Actual update (Linux)

# Building
npm run build                   # Production build
npm run build-storybook        # Static Storybook

# Linting & Type Checking
npm run lint                    # Check issues
npm run lint:fix               # Auto-fix ESLint issues
npm run typecheck              # TypeScript check (production code only)
npm run typecheck:stories      # TypeScript check (stories + components)
npm run typecheck:all          # TypeScript check (everything)

# Git & CI
./scripts/clean-local-screenshots.sh  # Clean local screenshots (macOS/Windows)
git checkout origin/main -- src/components/__visual__/__screenshots__/  # Reset screenshots
gh workflow run update-screenshots.yml  # Update visual baselines
gh run list --workflow=update-screenshots.yml --limit 1  # Check status

# Committing (pre-commit hooks run automatically)
git add .                        # Stage changes
git commit -m "feat: message"    # Husky runs lint-staged + screenshot check
# Pre-commit hooks:
#   1. Blocks screenshot commits (use GitHub Actions instead)
#   2. Runs typecheck:stories when .stories.tsx files are staged
#   3. Runs ESLint --max-warnings=0 on staged TS/TSX files
#   4. Runs Prettier on all staged files
# Commit message triggers auto-release:
#   feat: -> minor version bump (1.0.0 -> 1.1.0)
#   fix:  -> patch version bump (1.0.0 -> 1.0.1)
#   BREAKING CHANGE or !: -> major version bump (1.0.0 -> 2.0.0)

# Token System (v2.0.0+)
npm run test:compliance:run      # Verify token usage
grep -r "oklch(" src/styles/themes/  # Find hardcoded OKLCH (should be 0)
```

## Project Overview

A modern glassmorphism UI component library built with:

- **React 19.2** - Latest stable release with production-ready Server Components, enabling
  ahead-of-time rendering in separate environments for build-time or request-time execution
- **TypeScript 5.9** - Strict type checking for enhanced developer experience
- **Tailwind CSS 4.1** - CSS-first configuration with 5x faster full builds, 100x faster incremental
  builds (microseconds), automatic content detection, and CSS variables by default
- **Storybook 10.1** - ESM-only component workshop (29% smaller install), typesafe CSF factories,
  enhanced tag filtering, and native Vitest integration for testing
- **Vite 7** (rolldown-vite) - Rust-based Rolldown bundler providing 3-16x faster builds, 100x
  memory reduction, and unified dev/prod bundling
- **Vitest 4.0** - Stable browser mode with visual regression testing via `toMatchScreenshot`,
  Playwright traces for CI debugging, and first-class viewport testing

See [DEPENDENCIES.md](docs/technical/DEPENDENCIES.md) for detailed dependency documentation.

## Common Tasks for AI

### Adding a new Glass component

1. Create component in `src/components/glass/ui/[name]-glass.tsx`
2. Add variant definition in `src/lib/variants/[name]-glass-variants.ts`
3. Add unit tests in `src/components/glass/ui/__tests__/[name]-glass.test.tsx`
4. Add visual tests in `src/components/__visual__/[name].visual.test.tsx`
5. Add story in `src/components/glass/ui/[name]-glass.stories.tsx`
6. Update screenshots via GH workflow: `gh workflow run update-screenshots.yml`
7. Update registry: `npm run generate:registry`

### Adding a composite component

1. Create component in `src/components/glass/composite/[name]-glass.tsx`
2. Add unit tests in `src/components/glass/composite/__tests__/[name]-glass.test.tsx`
3. Add visual tests in `src/components/__visual__/[name].visual.test.tsx`
4. Add story in `src/components/glass/composite/[name]-glass.stories.tsx`
5. Update screenshots via GH workflow: `gh workflow run update-screenshots.yml`
6. Update registry: `npm run generate:registry`

### Fixing a visual regression test

- **DO NOT** update screenshots locally on macOS
- **DO** use GitHub Actions: `gh workflow run update-screenshots.yml`
- Reference screenshots are Linux-only (ubuntu-latest)
- After workflow completes: `git pull origin main`

### Migrating a component to compound API

- See [docs/migration/compound-components-v2.md](docs/migration/compound-components-v2.md) for
  pattern
- Use compound component API (ModalGlass.Root, ModalGlass.Content, etc.)
- Add both usage patterns to Storybook
- Document API in component JSDoc

### Adding a new dropdown component

- **Read first:** [docs/DROPDOWN_ARCHITECTURE.md](docs/DROPDOWN_ARCHITECTURE.md)
- Use Radix UI primitives (`DropdownMenu` or `Popover`)
- Import utilities from `src/lib/variants/dropdown-content-styles.ts`:
  - `getDropdownContentStyles()` - Glass surface styling
  - `dropdownContentClasses` - Container classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artyhoo/shadcn-glass-ui-library](https://github.com/artyhoo/shadcn-glass-ui-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->

---
trigger: always_on
description: Always follow these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# shipyard - GitHub Copilot Instructions

Always follow these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap, Build, and Test the Repository
Run commands in this exact sequence from the repository root:

- `npm run setup` -- installs all dependencies and runs allowed lifecycle scripts. Takes ~25 seconds. NEVER CANCEL. This project uses `@lavamoat/allow-scripts` to disable npm lifecycle scripts by default for security.
- `npx biome ci .` -- runs linting checks. Takes <1 second.
- `npm run test --workspaces --if-present` -- runs all unit tests. Takes <2 seconds total.
- `npm run build --workspace=demo` -- builds demo app. Takes ~4 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
- `npm run build --workspace=docs` -- builds docs app. Takes ~4 seconds. NEVER CANCEL. Set timeout to 120+ seconds.

### Run Unit Tests
Unit tests use Vitest and are located in individual packages:
- `cd packages/docs && npx vitest run` -- runs docs package tests (~1 second)
- `cd packages/base && npx vitest run` -- runs base package tests (~1 second)

### Run the Applications
Both demo and docs apps can run in development or preview mode:

**Development mode** (with hot reload):
- Demo: `cd apps/demo && npm run dev` -- starts at http://localhost:4321/, takes ~1 second to start
- Docs: `cd apps/docs && npm run dev` -- starts at http://localhost:4321/, takes ~1 second to start

**Preview mode** (production build):
- Demo: `cd apps/demo && npm run preview` -- requires build first, starts at http://localhost:4321/
- Docs: `cd apps/docs && npm run preview` -- requires build first, starts at http://localhost:4321/

### E2E Testing with Playwright
E2E tests are located in `apps/demo/tests/e2e/`:
- `cd apps/demo && npx playwright install --with-deps` -- installs browsers. Takes 5-15 minutes. NEVER CANCEL. Set timeout to 1200+ seconds. May fail in some environments due to download restrictions.
- `npm run test:e2e` -- runs E2E tests. Takes ~30 seconds if browsers are installed. NEVER CANCEL. Set timeout to 300+ seconds.
- `npm run test:e2e:ui` -- runs E2E tests with UI for debugging

Note: If Playwright browser installation fails, document it as "E2E tests require Playwright browsers which may fail to install in restricted environments."

## Validation
Always validate changes by running through complete end-to-end scenarios:

### Manual Validation Scenarios
After making any changes, ALWAYS test these scenarios:

1. **Build Validation**: Run both demo and docs builds successfully
2. **Navigation Testing**: 
   - Start demo app: `cd apps/demo && npm run dev`
   - Visit http://localhost:4321/ (redirects to locale-specific URL)
   - Test navigation between Blog, About pages
   - Verify internationalization works by visiting `/en/blog` and `/de/blog`
   - Confirm navigation links include locale prefixes correctly
3. **Content Validation**:
   - Verify blog posts load correctly
   - Check that titles display properly (format: "Site Title - Page Title")
   - Ensure styling with Tailwind CSS + DaisyUI works

### Pre-commit Validation
Always run these commands before committing changes or the CI (.github/workflows/checks.yml) will fail:
- `npx biome format --write .` -- formats code
- `npx biome ci .` -- validates formatting and linting

### Changeset Management
ALWAYS create changeset files for any package changes:
- `npx changeset add` -- creates a changeset for versioning and release notes
- Use appropriate semver levels: `patch` for bug fixes, `minor` for features, `major` for breaking changes
- Write user-focused descriptions (not implementation details)

## Repository Structure

### Monorepo Layout
```
shipyard/
├── packages/
│   ├── base/          # @levino/shipyard-base - Core components and layouts
│   ├── docs/          # @levino/shipyard-docs - Documentation features
│   └── blog/          # @levino/shipyard-blog - Blog functionality
├── apps/
│   ├── demo/          # Demo application showcasing all features
│   └── docs/          # Documentation site for the framework
├── .github/workflows/ # CI/CD with linting, unit tests, and E2E tests
└── .changeset/        # Version management and release notes
```

### Key Technology Stack
- **Framework**: Astro 5.x with TypeScript (strict mode)
- **Styling**: Tailwind CSS with DaisyUI components
- **Package Manager**: npm with workspaces
- **Testing**: Vitest (unit tests), Playwright (E2E tests)
- **Linting/Formatting**: Biome (replaces ESLint + Prettier)
- **Internationalization**: Built-in i18n with EN/DE locales
- **Git Hooks**: Lefthook with pre-commit formatting

### Package Dependencies
Each package uses peer dependencies for core frameworks:
- `astro: ^5` (Astro framework)
- `tailwindcss: ^3` (CSS framework) 
- `daisyui: ^4` (Component library)
- `@tailwindcss/typography: ^0.5.10` (Typography plugin)

## Common Tasks

### Frequently Used Commands and Outputs

#### Repository Root Contents
```bash
ls -la
```
```
.changeset/     # Changeset configuration for versioning
.github/        # GitHub workflows and configurations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levino/shipyard](https://github.com/levino/shipyard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->

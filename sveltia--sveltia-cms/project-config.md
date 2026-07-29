---
trigger: always_on
description: **Sveltia CMS** is a modern, Git-based headless content management system built as a drop-in replacement for Netlify/Decap CMS. The project is written in Svelte 5 with JavaScript, using Vite 8 as the build tool and Vitest 4 for testing.
---

# Copilot Instructions: Sveltia CMS

## Repository Overview

**Sveltia CMS** is a modern, Git-based headless content management system built as a drop-in replacement for Netlify/Decap CMS. The project is written in Svelte 5 with JavaScript, using Vite 8 as the build tool and Vitest 4 for testing.

**Key Details:**

- **Size**: ~800 source files, 225+ test files, 6,200+ tests
- **Languages**: TypeScript-flavoured JavaScript (ES2024/JSDoc), Svelte 5, SCSS/CSS, HTML
- **Target Runtime**: Browser (IIFE and ES modules)
- **Package Manager**: **pnpm only** (npm will cause issues)
- **Node Version**: v25 (see `.nvmrc`)
- **Bundle Output**: Distributable CMS that loads in browsers via CDN or npm

## Build & Development Commands

### Prerequisites

**CRITICAL**: Always use `pnpm` - `npm` will not work correctly with this project.

```bash
# Install pnpm if not available
npm install -g pnpm@latest

# Install dependencies (always run first)
pnpm install
```

### Core Development Commands

```bash
# Development server with hot reload
pnpm dev

# Production build (creates package/dist/ directory)
pnpm build

# Watch build for development
pnpm build:watch

# Preview production build
pnpm preview
```

### Quality Assurance Commands

```bash
# Run ALL checks (recommended before committing)
pnpm check

# Individual checks
pnpm check:eslint    # ESLint (JavaScript/Svelte linting)
pnpm check:prettier  # Code formatting
pnpm check:stylelint # CSS/SCSS linting
pnpm check:svelte    # Svelte compiler checks
pnpm check:oxlint    # Additional fast linting
pnpm check:cspell    # Spell checking
pnpm check:imports   # Unused import detection
pnpm check:audit     # Security audit

# Testing
pnpm test                # Run all tests
pnpm test:coverage       # Test coverage report

# Code formatting
pnpm format          # Auto-fix Prettier formatting
```

### Common Issues & Solutions

**pnpm not found**: Install with `npm install -g pnpm@latest`

**Build failures**: Always run `pnpm install` first after any `package.json` changes

**Import errors**: Use the custom `find-unused-imports.js` script via `pnpm check:imports` - it’s more accurate than standard tools for this project

**Svelte 5 compatibility**: This project uses Svelte 5 with runes - ensure any Svelte code follows the new syntax patterns

## Project Architecture & Layout

### Source Structure

```
src/lib/
├── components/          # Svelte UI components
│   ├── app.svelte      # Main app component
│   ├── assets/         # Asset management UI
│   ├── contents/       # Content editing UI
│   └── ...
├── services/           # Business logic & data services
│   ├── app/           # Core app services
│   ├── assets/        # Asset management
│   ├── backends/      # Git backend integrations (GitHub, GitLab, Gitea)
│   ├── config/        # CMS configuration handling
│   ├── contents/      # Content & collection management
│   ├── integrations/  # External service integrations
│   ├── user/          # User authentication & preferences
│   └── utils/         # Utility functions
├── types/             # JavaScript and TypeScript type definitions
├── locales/           # Internationalization files
└── main.js            # Entry point
```

### Key Configuration Files

- `vite.config.js`: Build configuration with custom plugins
- `svelte.config.js`: Svelte 5 with runes enabled
- `jsconfig.json`: JavaScript & path mapping (`$lib/*`)
- `eslint.config.js`: Comprehensive ESLint rules with Svelte plugin (flat config format)
- `.prettierrc.yaml`: Code formatting (single quotes, trailing commas)
- `.stylelintrc.yaml`: SCSS/CSS linting rules
- `package.json`: Scripts and dependencies
- `.nvmrc`: Node v25 requirement

### Build Output

- `package/dist/sveltia-cms.js`: IIFE bundle for browser `<script>` tag
- `package/dist/sveltia-cms.mjs`: ES module for npm consumers
- `package/`: Complete npm package directory with types

## GitHub CI/CD Pipeline

**Workflow**: `.github/workflows/tests.yml`

- **Triggers**: Every push to any branch
- **Jobs**: Check, Test, Build (run in parallel matrix)
- **Node**: Uses `.nvmrc` version (v25)
- **Package Manager**: pnpm with cache
- **Steps**: checkout → setup → install → run task

**Validation Steps for PRs:**

1. All ESLint rules pass (strict Airbnb config + Svelte rules + some customizations)
2. Prettier formatting enforced
3. All 6000+ tests pass
4. Svelte compiler checks pass
5. Production build succeeds
6. No unused imports (custom script validation)

## Development Guidelines

### Coding Standards

- **Guidelines**: Follow Airbnb JavaScript style guide with project-specific overrides (see `eslint.config.js`)
- **Quotes**: Single quotes for JavaScript, double for YAML/CSS (see `.prettierrc.yaml`)
- **Line Length**: 100 characters max
- **Trailing Commas**: Always use
- **Import Sorting**: Automatic via ESLint (builtin → external → internal → $lib)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sveltia/sveltia-cms](https://github.com/sveltia/sveltia-cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

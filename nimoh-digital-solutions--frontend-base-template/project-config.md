---
trigger: always_on
description: **TAST** (Template for Application Starter Toolkit) is a production-ready React starter kit and monorepo that ships a demo application alongside a suite of publishable packages. It is built with:
---

# AGENTS.md

## Project Overview

**TAST** (Template for Application Starter Toolkit) is a production-ready React starter kit and monorepo that ships a demo application alongside a suite of publishable packages. It is built with:

| Layer | Technology |
|---|---|
| Framework | React 19 + TypeScript (strict) |
| Build | Vite 7, SWC |
| Styling | SCSS Modules + Open Props (design tokens) |
| State | Zustand, React Hook Form + Zod |
| Routing | React Router v7 |
| i18n | i18next / react-i18next |
| Testing | Vitest + React Testing Library + jest-axe |
| Linting | ESLint 8, Stylelint 16, Prettier |
| Commits | Husky + lint-staged + Commitlint (Conventional Commits) |
| CI / Release | GitHub Actions, Changesets |
| PWA | vite-plugin-pwa (opt-in) |
| Container | Docker + nginx |
| Package Mgr | Yarn 4 (Berry) with workspaces |

Publishable packages live under `packages/` and are scoped to `@nimoh-digital-solutions`:

| Package | Purpose |
|---|---|
| `tast-ui` | Shared React component library (Storybook) |
| `tast-hooks` | Reusable custom hooks |
| `tast-utils` | General-purpose utilities |
| `tast-styles` | Shared SCSS tokens & mixins |
| `create-tast-app` | CLI scaffolding tool |
| `eslint-config` | Shared ESLint config |
| `stylelint-config` | Shared Stylelint config |
| `tsconfig` | Shared TypeScript base configs |

---

## Repository Structure

```
├── .github/
│   ├── agents/          # Copilot agent definitions (not committed — .gitignore)
│   ├── instructions/    # Copilot instruction files (not committed — .gitignore)
│   └── workflows/       # CI & Release GitHub Actions (committed)
├── packages/            # Publishable Yarn workspace packages
│   ├── tast-ui/         # Component library
│   ├── tast-hooks/      # Custom hooks
│   ├── tast-utils/      # Utility functions
│   ├── tast-styles/     # SCSS design tokens
│   ├── create-tast-app/ # CLI scaffolding
│   ├── eslint-config/   # Shared ESLint rules
│   ├── stylelint-config/# Shared Stylelint rules
│   └── tsconfig/        # Shared TS configs
├── src/                 # Demo / starter application
│   ├── assets/          # Static assets (images, SVGs)
│   ├── components/      # Shared UI components
│   ├── configs/         # App-level configuration
│   ├── contexts/        # React context providers
│   ├── data/            # Static data / constants
│   ├── features/        # Feature-based modules
│   ├── hooks/           # App-specific custom hooks
│   ├── i18n/            # Internationalization resources
│   ├── layouts/         # Page layout components
│   ├── pages/           # Route-level page components
│   ├── routes/          # Route definitions
│   ├── services/        # API / HTTP service layer
│   ├── styles/          # Global SCSS & design tokens
│   ├── sw/              # Service worker (PWA)
│   ├── test/            # Test setup & helpers
│   ├── types/           # Shared TypeScript types
│   └── utils/           # App-specific utilities
├── plugins/             # Custom Vite plugins
├── scripts/             # Setup & scaffolding scripts
├── public/              # Static public assets & PWA manifest
└── api/                 # API proxy / mock helpers
```

---

## Development Workflow

### Getting Started

```bash
# 1. Install dependencies (Yarn 4 — corepack enable first)
yarn install

# 2. Start the dev server
yarn dev

# 3. Run the full check suite
yarn check          # audit + type-check + lint + stylelint + test
```

### Key Commands

| Command | Description |
|---|---|
| `yarn dev` | Start Vite dev server |
| `yarn build` | TypeScript check + production build |
| `yarn test` | Run Vitest in watch mode |
| `yarn test:run` | Single Vitest run |
| `yarn test:coverage` | Vitest with coverage report |
| `yarn lint` / `yarn lint:fix` | ESLint (src/) |
| `yarn stylelint` / `yarn stylelint:fix` | Stylelint (SCSS) |
| `yarn format` | Prettier (all files) |
| `yarn type-check` | `tsc --noEmit` |
| `yarn packages:build` | Build all publishable packages |
| `yarn storybook` | Launch tast-ui Storybook |
| `yarn changeset` | Create a new changeset for versioning |
| `yarn changeset:publish` | Build packages + publish via Changesets |

### Branching & Commits

- Default branch: **main**
- Commits follow **Conventional Commits** (enforced by Commitlint + Husky):
  `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`, `revert`
- Pre-commit hooks run **lint-staged** (ESLint fix, Stylelint fix, Prettier)

### CI Pipeline (`.github/workflows/ci.yml`)

Runs on push/PR to `main`:

1. Checkout → Enable Corepack → Setup Node 22 → `yarn install --immutable`
2. `yarn packages:build`
3. `yarn type-check`
4. `yarn lint` (ESLint)
5. `yarn stylelint` (Stylelint)
6. `yarn vitest run --coverage`

### Release Pipeline (`.github/workflows/release.yml`)

Runs on push to `main`:

1. Same setup as CI
2. Builds packages
3. Uses `changesets/action` to either:
   - Open a "Version Packages" PR when un-released changesets exist
   - Publish to npm when the Version Packages PR merges

---

## Coding Standards

### TypeScript

- Strict mode enabled — no `any` unless absolutely necessary

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimoh-Digital-Solutions/frontend-base-template](https://github.com/Nimoh-Digital-Solutions/frontend-base-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->

---
trigger: always_on
description: This document provides coding agents with essential information about the blog-1999 codebase.
---

# Agent Guidelines for blog-1999

This document provides coding agents with essential information about the blog-1999 codebase.

## Project Overview

Next.js (App Router) blog system with MDX content and Notion CMS integration. Uses TypeScript, Tailwind CSS v4, React 19, and server-side rendering. The toolchain is Nix-managed (Node 22 / pnpm 10).

## Build & Development Commands

```bash
# Install dependencies (uses pnpm 10)
pnpm install

# Development server (http://localhost:3000)
pnpm run dev

# Production build
pnpm run build

# Start production server
pnpm run start

# Lint all files (ESLint flat config)
pnpm run lint

# Auto-fix lint/formatting issues
pnpm run lint:fix

# Export static site
pnpm run export
```

### Nix (Blueprint layout)

The repo is Nix-managed via [numtide/blueprint](https://numtide.github.io/blueprint/),
with all Nix files under `nix/` (prefix configured in `flake.nix`).

```bash
nix develop        # reproducible dev shell (Node 22, pnpm 10, TypeScript) + installs git hooks
nix flake check    # runs lint + typecheck checks in an offline sandbox
nix fmt            # treefmt: prettier (project) + nixfmt (*.nix)
```

With [direnv](https://direnv.net), `.envrc` (`use flake`) auto-loads the dev
shell on `cd` (run `direnv allow` once).

- `nix/devshell.nix` - default dev shell; its shellHook installs the git-hooks.
- `nix/formatter.nix` + `nix/treefmt.nix` - `nix fmt` (shared treefmt config).
- `nix/pre-commit-check.nix` - [git-hooks.nix](https://github.com/cachix/git-hooks.nix)
  config (treefmt, eslint, tsc, nil, statix). Devshell-only (not a flake check),
  because eslint/tsc/prettier need the project-local `node_modules`. Runs on
  every `git commit`; the generated `.pre-commit-config.yaml` is gitignored.
- `nix/checks/{lint,typecheck}.nix` - flake checks (consume the deps package via
  `perSystem.self.pnpm-deps`). These provide the offline/CI lint+typecheck
  coverage that the working-tree git hooks cannot run in the sandbox.
- `nix/packages/pnpm-deps.nix` - offline pnpm dependency store, exposed as
  `packages.<system>.pnpm-deps` (a fixed-output derivation). **When
  `pnpm-lock.yaml` changes, the `hash` here must be regenerated**: set it to
  `pkgs.lib.fakeHash`, run a build, copy the `got:` hash.

### Testing

- No automated test suite currently configured
- Manual testing via `pnpm run dev` and browser verification
- Type checking via TypeScript compiler: `npx tsc --noEmit`
- CI-style verification: `nix flake check` (lint + typecheck, fully offline)

## Code Style & Formatting

### Prettier Configuration

- **Print Width**: 70 characters
- **Indentation**: 2 spaces, no tabs
- **Quotes**: Double quotes for strings
- **Semicolons**: Required
- **Trailing Commas**: Always (ES5+ compatible)
- **Arrow Functions**: Avoid parentheses when possible (`x => x`)
- **Bracket Spacing**: Enabled (`{ foo }` not `{foo}`)
- **JSX**: Brackets on same line as last prop
- **Plugins**: prettier-plugin-tailwindcss (auto-sorts classes)

### ESLint Rules

- **Flat config**: `eslint.config.mjs` (ESLint 9 flat config; the legacy
  `.eslintrc.json` has been removed)
- Composes: `@eslint/js` recommended, `eslint-config-next` (core-web-vitals +
  typescript), `typescript-eslint` recommended, and
  `eslint-plugin-prettier/recommended`
- **Unused vars**: Error
- **Explicit any**: Error (avoid `any` type)
- Prettier integration enabled
- Run directly with `pnpm run lint` (`eslint .`); `next lint` is removed in
  Next 16

## TypeScript Configuration

### Strict Mode

- `strict: false` (permissive mode)
- `strictNullChecks: false`
- Use optional chaining (`?.`) and nullish coalescing (`??`) liberally

### Path Aliases

```typescript
@/components/* → components/*
@/lib/*        → lib/*
@/data/*       → data/*
@/public/*     → public/*
```

### Target & Module

- Target: ES2017
- Module: esnext, bundler resolution (set by Next 16)
- JSX: react-jsx (React 17+ transform)

## Project Structure

```
app/               - Next.js 13+ App Router pages
  (root)/          - Main site pages with root layout
  (dyn)/           - Dynamic Notion-based content
components/        - React components
  UI/              - User interface components
  MDX/             - MDX content components
  Scripts/         - Analytics and scripts
  Layouts/         - Layout components
  Widgets/         - Reusable widgets
lib/               - Utilities and external service integrations
data/              - Static content (MDX blog posts, metadata)
  blog/            - MDX blog post files
public/            - Static assets (images, files, etc.)
styles/            - Global CSS, Prism themes, KaTeX styles
locales/           - i18n translations (en, zh)
```

## Naming Conventions

### Files & Directories

- **Components**: PascalCase (`WelcomeCard.tsx`, `PostPage.tsx`)
- **Utilities**: camelCase (`blog.ts`, `spotify.ts`)
- **Directories**: PascalCase for component folders, lowercase for utility folders
- **Route Segments**: Next.js conventions (`[lang]`, `(root)`, `layout.tsx`, `page.tsx`)

### Code

- **React Components**: PascalCase, default export
- **Functions**: camelCase
- **Types/Interfaces**: PascalCase
- **Constants**: camelCase (not SCREAMING_SNAKE_CASE)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whexy/blog-1999](https://github.com/whexy/blog-1999) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

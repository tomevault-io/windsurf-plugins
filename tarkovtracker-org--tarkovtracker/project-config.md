---
trigger: always_on
description: TarkovTracker is a Nuxt 4 Single Page Application (SPA) for tracking Escape from Tarkov game progress. The app runs client-side only (`ssr: false`) with Nitro server routes for API proxying.
---

# GitHub Copilot Instructions for TarkovTracker

## Project Overview

TarkovTracker is a Nuxt 4 Single Page Application (SPA) for tracking Escape from Tarkov game progress. The app runs client-side only (`ssr: false`) with Nitro server routes for API proxying.

**Tech Stack:**

- Nuxt 4 with Vue 3 Composition API
- TypeScript (strict mode)
- Pinia for state management
- Supabase for auth, database, and realtime
- Cloudflare Pages/Workers for deployment
- Tailwind CSS v4 (no SCSS/scoped styles)
- Vitest + Vue Test Utils for testing

## Directory Structure

```
app/                    # Nuxt 4 source directory
├── pages/              # File-based routing (kebab-case)
├── features/           # Domain slices (tasks, team, hideout, maps, neededitems, traders, settings, admin)
├── components/         # Shared UI components
├── shell/              # App chrome (AppBar, NavDrawer, AppFooter)
├── stores/             # Pinia stores with persistence
├── composables/        # Reusable composition functions
├── server/api/         # Nitro server routes
├── locales/            # i18n JSON files (en, de, es, fr, ru, uk)
├── plugins/            # Nuxt plugins
├── utils/              # Utility functions
└── types/              # TypeScript type definitions
supabase/functions/     # Deno-based Edge Functions (different lint rules)
workers/                # Cloudflare Workers (api-gateway)
```

## Commands

**Prerequisites:** Node.js >=24.12.0 and npm >=11.6.2 (see `package.json` engines). No additional environment variables are required for local development.

```bash
npm run dev             # Dev server at localhost:3000
npm run build           # Production SPA build
npm run lint            # ESLint with zero warnings (enforced by CI/pre-commit hooks)
npm run lint:fix        # Auto-fix lint issues
npm run format          # Prettier + ESLint fix (run before lint to auto-fix formatting)
npx vitest              # Run unit tests
npx vitest --ui         # Tests with dashboard
```

**Recommended workflow:** Run `npm run format` before `npm run lint` to auto-fix formatting issues. The lint step enforces zero warnings as a hard requirement in CI and pre-commit hooks.

## Code Style Requirements

### Formatting (Prettier)

- 2-space indentation
- 100-character line width
- Single quotes for strings
- Semicolons required
- Trailing commas (es5 style)
- LF line endings

### Vue Single File Components

Always use this structure:

```vue
<script setup lang="ts">
  // Indented script content (vueIndentScriptAndStyle: true)
</script>

<template>
  <!-- Template content with Tailwind classes -->
</template>
```

**No `<style>` blocks.** Use Tailwind CSS v4 utilities exclusively. For complex animations or utilities not available in Tailwind, add them to `app/assets/css/tailwind.css`.

### Import Rules (Critical)

1. **Never use parent-relative imports** - Use `@/` or `~/` aliases instead

   ```typescript
   // WRONG
   import { foo } from '../../utils/foo';

   // CORRECT
   import { foo } from '@/utils/foo';
   ```

2. **No blank lines between import groups**
3. **Alphabetically sorted imports** (case-insensitive)
4. **Import group order:** builtin → external → internal → parent → sibling → index → object → type

### TypeScript Guidelines

- Prefer explicit types for exported functions, stores, and composables
- Avoid `any`; use `unknown` with type narrowing
- Use union/string literal types for constrained values
- Use `as const` for literal inference
- Keep types close to usage; reuse existing types

### Naming Conventions

- **Components:** PascalCase (`TaskCard.vue`)
- **Composables:** camelCase with `use` prefix (`useTaskFiltering.ts`)
- **Stores:** `useXStore` pattern (`useProgress.ts`)
- **Routes/files:** kebab-case (`needed-items.vue`)
- **Test files:** `*.test.ts` in `__tests__/` folders
- **Constants:** UPPER_SNAKE_CASE for globals

## State Management (Three-Store Architecture)

```typescript
// useMetadata - Static game data from tarkov.dev API
const metadata = useMetadata();

// useProgress - User progress state (completions, objectives)
const progress = useProgress();

// usePreferences - User settings with localStorage persistence
const preferences = usePreferences();
```

## Key Patterns

### Dual Game Mode

The app tracks PvP and PvE progress separately. Always consider both modes when working with progress data.

### Tarkov.dev Import and Linking

- Persist a single linked `tarkovUid` only.
- Do not add or depend on a persisted linked-mode/imported-mode field.
- Treat PvP/PvE choice for tarkov.dev imports as temporary UI state that decides where imported
  progress is written.
- Accept full tarkov.dev player profile URLs for imports, fetch profile JSON through the local
  `/api/tarkov-dev/profile` route, and reuse the canonical Tarkov.dev profile parser.
- Remind users to open their tarkov.dev profile page before importing because that page visit
  refreshes the public profile JSON.
- Build tarkov.dev profile links from the currently viewed or selected mode to choose the
  `regular` or `pve` URL slug.

### Team System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarkovtracker-org/TarkovTracker](https://github.com/tarkovtracker-org/TarkovTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

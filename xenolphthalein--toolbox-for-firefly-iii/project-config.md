---
trigger: always_on
description: Toolbox for Firefly III is a web application that extends Firefly III (personal finance manager) with automation tools. It provides features like duplicate detection, AI-powered category/tag suggestions, subscription pattern detection, and Amazon/PayPal transaction matching.
---

# AGENTS.md — Toolbox for Firefly III Development Guide

## Project Overview

Toolbox for Firefly III is a web application that extends Firefly III (personal finance manager) with automation tools. It provides features like duplicate detection, AI-powered category/tag suggestions, subscription pattern detection, and Amazon/PayPal transaction matching.

---

## Tech Stack

### Frontend
- **Vue 3** with Composition API (`<script setup>`)
- **Vuetify 3** for UI components
- **TypeScript** for type safety
- **Vite** for build tooling

### Backend
- **Express.js** with TypeScript
- **Server-Sent Events (SSE)** for streaming progress updates
- **Session-based state** for multi-step operations (Amazon/PayPal uploads)

### Shared
- Common TypeScript types in `src/shared/types/`

---

## Project Structure

```
src/
├── client/                 # Vue frontend
│   ├── components/
│   │   ├── common/         # Reusable components (WizardStepper, DateRangeStep, etc.)
│   │   └── layout/         # App shell (AppNavigation, AppHeader)
│   ├── composables/        # Vue composables (useProgress, useStreamProcessor, etc.)
│   ├── views/              # Page components (*View.vue)
│   ├── stores/             # Pinia stores
│   ├── services/           # API client
│   └── utils/              # Formatting helpers
├── server/                 # Express backend
│   ├── routes/             # API route handlers
│   ├── services/           # Business logic
│   └── utils/              # Server utilities
└── shared/
    └── types/              # Shared TypeScript interfaces
```

---

## Key Patterns

### Tool Views (DuplicatesView, CategoriesView, etc.)
All tool views follow a consistent wizard pattern:
1. **Step 1**: Date range selection with transaction preview
2. **Step 2+**: Analysis/matching with streaming progress
3. **Results**: Summary card with selection and bulk actions

Use these shared components:
- `DateRangeStep` - Date picker + transaction preview
- `ResultsSummaryCard` - Stats chips + select all + action button
- `FinalActionButton` - Primary action that toggles to "Re-run" after first use
- `FileUploadCard` - File input with preview (Amazon/PayPal)
- `ProgressCard` - Progress bar during streaming operations

### Composables
- `useStreamProcessor` - SSE stream handling (replaces ~40 lines of boilerplate)
- `useProgress` - Progress state (current, total, message)
- `useSelection` - Multi-select with select all
- `useTransactionPreview` - Paginated transaction loading

### Streaming Pattern
Backend endpoints use SSE for long-running operations:
```typescript
// Server sends events like:
{ type: 'progress', data: { current: 5, total: 100, message: '...' } }
{ type: 'result', data: { /* result item */ } }
{ type: 'complete', data: null }
```

Frontend uses `useStreamProcessor`:
```typescript
const { processStream } = useStreamProcessor();
await processStream('/api/endpoint', body, handleStreamEvent);
```

---

## Conventions

### Vue Components
- Use `<script setup lang="ts">` syntax
- Props with `defineProps<T>()` and `withDefaults()`
- Emit with `defineEmits<T>()`
- All cards use `rounded="lg"`

### Styling
- Use Vuetify utility classes (`d-flex`, `align-center`, `ga-4`, `mb-4`, etc.)
- Scoped styles only when necessary
- No external CSS frameworks

### TypeScript
- Strict mode enabled
- Explicit types for props, emits, and function parameters
- Shared types in `@shared/types/app`

### Naming
- Components: `PascalCase.vue`
- Composables: `useCamelCase.ts`
- Views: `*View.vue`
- Routes: kebab-case (`/api/duplicates/stream-find`)

---

## Commands

```bash
npm run dev          # Start dev server (frontend + backend)
npm run build        # Production build
npm run typecheck    # TypeScript validation (run after changes)
npm run lint         # ESLint check
npm run format       # Prettier formatting
npm run i18n:check   # Check for duplicate translation keys
```

**Always run `npm run typecheck` and `npm run lint` after making changes.**

---

## Common Tasks

### Adding a New Tool View
1. Create `src/client/views/NewToolView.vue` following existing patterns
2. Add route in `src/client/router/index.ts`
3. Add navigation item in `AppNavigation.vue`
4. Create backend routes in `src/server/routes/`

### Creating a Reusable Component
1. Create in `src/client/components/common/`
2. Export from `src/client/components/common/index.ts`
3. Use `defineProps` with JSDoc comments for documentation

### Adding a Composable
1. Create in `src/client/composables/`
2. Export from `src/client/composables/index.ts`
3. Return reactive refs and functions

### Working with Internationalization (i18n)

The application uses **vue-i18n** for multi-language support. Currently supports English (`en`) and German (`de`).

**Using translations in components:**
```vue
<script setup lang="ts">
import { useI18n } from 'vue-i18n';

const { t } = useI18n();
</script>

<template>
  <v-btn>{{ t('common.buttons.save') }}</v-btn>
  <h1>{{ t('duplicates.title') }}</h1>
</template>
```

**Translation file structure:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xenolphthalein/toolbox-for-firefly-iii](https://github.com/Xenolphthalein/toolbox-for-firefly-iii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

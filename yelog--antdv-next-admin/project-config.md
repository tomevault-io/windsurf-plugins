---
trigger: always_on
description: A Vue 3 + TypeScript + Ant Design Vue admin scaffold with RBAC, theming, i18n, and mock APIs.
---

# Antdv Next Admin - Agent Guidelines

A Vue 3 + TypeScript + Ant Design Vue admin scaffold with RBAC, theming, i18n, and mock APIs.

## Project Structure

```
src/
├── api/              # API layer - organized by domain (auth.ts, user.ts)
├── assets/styles/    # Global styles (variables.css, animations.css, global.css)
├── components/       # Reusable components (Layout/, Permission/, etc.)
├── composables/      # Composition functions (usePermission.ts, useWatermark.ts)
├── directives/       # Custom Vue directives (permission.ts)
├── locales/          # i18n translations (zh-CN.ts, en-US.ts)
├── router/           # Vue Router (routes.ts, guards.ts, utils.ts)
├── stores/           # Pinia stores - one per domain (auth.ts, theme.ts, layout.ts)
├── types/            # TypeScript interfaces/types (auth.ts, api.ts, router.ts)
├── utils/            # Pure utility functions (request.ts, storage.ts, helpers.ts)
└── views/            # Page components (dashboard/, system/, examples/)

mock/
├── data/             # Mock datasets (users.data.ts, roles.data.ts)
└── handlers/         # Mock API handlers (auth.mock.ts, user.mock.ts)

tests/
├── e2e/              # End-to-end tests (*.spec.ts) - templates for future Playwright setup
└── unit/             # Unit tests (*.spec.ts) - templates for future Vitest setup
```

## Build, Test, and Development Commands

### Essential Commands
```bash
npm install              # Install all dependencies
npm run dev              # Start dev server at http://localhost:3000 (with mock APIs)
npm run build            # Type check + production build → dist/
npm run preview          # Preview production build locally
npm run type-check       # Run vue-tsc --noEmit (NO auto-fix)
```

### Pre-commit Requirements
**BEFORE any commit or PR:**
1. Run `npm run type-check` - must exit 0 with no errors
2. Run `npm run build` - must complete successfully
3. For RBAC/auth changes: manually verify login with `admin/123456` and `user/123456`

### Testing Notes
- **No test runner configured yet** - Playwright/Vitest dependencies are NOT installed
- Test files in `tests/` are **templates** for future setup
- To add tests later: install test framework first, update package.json scripts, then write tests

## Code Style Guidelines

### Formatting (EditorConfig)
- **Indentation**: 2 spaces (NO tabs)
- **Line endings**: LF (Unix-style)
- **Encoding**: UTF-8
- **Final newline**: required
- **Trailing whitespace**: trimmed (except in .md files)

### TypeScript
- **Strict mode enabled** (`tsconfig.json`): all strict checks ON
- **Path aliases**: use `@/` for `src/` (e.g., `import { useAuthStore } from '@/stores/auth'`)
- **Type annotations**: explicit return types for public functions/composables
- **Type definitions**: place shared types in `src/types/`, domain-specific types near usage
- **No type suppression**: NEVER use `as any`, `@ts-ignore`, or `@ts-expect-error`

### Vue Component Style
**Component naming:**
- **Files**: PascalCase for reusable components (`NotificationPanel.vue`, `ThemeToggle.vue`)
- **Views**: route-based folders with `index.vue` (`src/views/dashboard/index.vue`)

**Component structure (Composition API only):**
```vue
<template>
  <!-- Template using script setup's reactive state -->
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useAuthStore } from '@/stores/auth'
import type { User } from '@/types/auth'

// Props
interface Props {
  userId: string
  mode?: 'edit' | 'view'
}
const props = withDefaults(defineProps<Props>(), {
  mode: 'view'
})

// Emits
const emit = defineEmits<{
  save: [user: User]
  cancel: []
}>()

// State
const authStore = useAuthStore()
const loading = ref(false)
const user = computed(() => authStore.user)

// Methods (prefer explicit function declarations)
function handleSave() {
  // Implementation
}
</script>

<style scoped>
/* Component-specific styles */
</style>
```

### Import Ordering
Group imports in this order (blank line between groups):
```ts
// 1. Vue core
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'

// 2. Third-party libraries
import { message } from 'antdv-next'
import dayjs from 'dayjs'

// 3. Project imports (@/ alias)
import { useAuthStore } from '@/stores/auth'
import { login, getUserInfo } from '@/api/auth'
import type { User, LoginParams } from '@/types/auth'
```

### Naming Conventions
| Type | Convention | Example |
|------|-----------|---------|
| Components | PascalCase | `NotificationPanel.vue`, `TabBar.vue` |
| Composables | `useXxx.ts` | `usePermission.ts`, `useFullscreen.ts` |
| Stores | Domain-based | `auth.ts`, `permission.ts`, `theme.ts` |
| Types/Interfaces | PascalCase | `User`, `LoginParams`, `ApiResponse<T>` |
| Functions | camelCase | `getUserInfo()`, `checkPermission()` |
| Constants | SCREAMING_SNAKE_CASE | `TOKEN_KEY`, `API_BASE_URL` |

### Error Handling
- **Try/catch**: wrap all async operations with meaningful error messages
- **Axios interceptors**: global error handling in `src/utils/request.ts`
- **User feedback**: use `message.error()` or `notification.error()` from antdv-next
```ts
try {
  const response = await getUserInfo()
  // Success path
} catch (error) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yelog/antdv-next-admin](https://github.com/yelog/antdv-next-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->

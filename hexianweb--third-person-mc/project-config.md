---
trigger: always_on
description: This document provides essential information for AI coding agents working in this repository.
---

# AGENTS.md - Development Guide for AI Coding Agents

This document provides essential information for AI coding agents working in this repository.

---

## Build, Lint, and Test Commands

### Package Manager

This project uses **pnpm**. Always use `pnpm` instead of `npm` or `yarn`.

### Development

```bash
pnpm dev              # Start development server with hot reload
pnpm build            # Build for production
pnpm preview          # Preview production build locally
```

### Linting

```bash
pnpm lint             # Check code for linting errors
pnpm lint:fix         # Auto-fix linting issues
```

### Testing

```bash
# E2E tests with Playwright (located in tests/ directory)
pnpm test:chrome      # Run tests in Chromium (headed mode)
pnpm test:firefox     # Run tests in Firefox (headed mode)
pnpm test:safari      # Run tests in WebKit/Safari (headed mode)
```

**Running a single test:**

```bash
npx playwright test tests/browsers.test.js --headed --project=chromium
npx playwright test tests/browsers.test.js:10 --headed  # Run specific line
```

### Cleanup

```bash
pnpm clean:dist       # Remove dist directory
pnpm clean:report     # Remove Playwright test reports
```

---

## Code Style Guidelines

### Language & Modules

- **Pure JavaScript** (no TypeScript) with ES modules (`"type": "module"`)
- Use `.js` extensions explicitly in imports
- Use JSDoc comments for type hints where beneficial

### Formatting (ESLint - Antfu)

- **Style Guide:** Follows `@antfu/eslint-config` defaults
- **Print width:** 120 characters
- **Indentation:** 2 spaces (no tabs)
- **Semicolons:** None (Never use semicolons)
- **Quotes:** Single quotes (`'`) for JS/Vue, single for JSX
- **Trailing commas:** All (Multi-line objects/arrays)
- **Arrow parens:** Always
- **Line endings:** CRLF (Windows)
- **Plugins:** `prettier-plugin-tailwindcss` for class sorting

**CRITICAL FORMATTING RULES:**

1. **Respect Existing Style:** When modifying a file, mimic the existing formatting (braces, blank lines, sorting) exactly.
2. **Vue SFC:** Do NOT indent top-level content inside `<script setup>` or `<style>`.
3. **Minimal Changes:** Do NOT reformat the entire file. Only apply formatting to your specific changes.
4. **Antfu Nuances:** Allow single-line `if` statements without braces.

### Linting (ESLint)

- Uses **@antfu/eslint-config** (opinionated, comprehensive)
- Additional plugins:
  - `eslint-plugin-prettier` - Format via Prettier
  - `eslint-plugin-import` - Validate imports
  - `eslint-plugin-simple-import-sort` - Auto-sort imports
  - `eslint-plugin-sonarjs` - Code quality rules
  - `eslint-plugin-unicorn` - Modern JS best practices
  - `eslint-plugin-promise` - Promise patterns

### Naming Conventions

- **Classes:** PascalCase (`Experience`, `ChunkManager`, `Camera`)
- **Functions/Methods:** camelCase (`updateCamera`, `debugInit`, `resize`)
- **Private members:** Underscore prefix (`_adaptiveY`, `_internalState`)
- **Constants:** UPPER_SNAKE_CASE (`SEED_MAX`, `CHUNK_SIZE`)
- **Vue Components:** PascalCase file names (`Crosshair.vue`, `MiniMap.vue`)
- **Pinia Stores:** camelCase with `use` prefix (`useUiStore`, `useHudStore`)

### Import Order

1. External packages (Three.js, Vue, Pinia, etc.)
2. Internal modules (from `src/`)
3. Assets (shaders, textures, models)

Example:

```javascript
import i18n from '@three/i18n.js'
import { createPinia } from 'pinia'
import { createApp } from 'vue'
import App from './App.vue'
import '@styles/main.scss'

const app = createApp(App)
app.use(createPinia())
app.use(i18n)
app.mount('#app')
```

### Type Annotations

Use JSDoc for complex types:

```javascript
/**
 * @param {THREE.Vector3} position - Player position
 * @param {number} radius - Detection radius
 * @returns {Array<THREE.Object3D>} Nearby entities
 */
function findNearbyEntities(position, radius) { ... }
```

### Error Handling

- Use try-catch for async operations and critical paths
- Log errors with context: `console.error('Failed to load texture:', error)`
- Provide fallbacks for non-critical failures
- Validate inputs for public APIs

### Comments

- Use **Chinese comments** for complex logic (this is a bilingual codebase)
- Document "why" not "what" for non-obvious code
- Add JSDoc for all public classes and methods
- Include inline comments for tricky algorithms

Example:

```javascript
/**
 * 计算环境光遮蔽强度
 * Calculates ambient occlusion intensity
 */
calculateAO(vertices) {
  // 使用顶点法线计算遮蔽因子
  // Calculate occlusion factor using vertex normals
  ...
}
```

---

## Architecture Patterns

### Singleton Pattern (Three.js Core)

All Three.js logic is managed through the `Experience` singleton:

```javascript
import Experience from './experience.js'

class MyComponent {
  constructor() {
    this.experience = new Experience() // Returns singleton
    this.scene = this.experience.scene
    this.camera = this.experience.camera
    // Only extract what this component needs
  }
}
```

**Rules:**

- Access core systems via `this.experience`
- Only extract necessary dependencies (avoid coupling)
- All 3D components must be class-based

### State Management (Pinia)

Use Pinia for global state shared between UI and 3D logic:

```javascript
// In store (src/pinia/uiStore.js)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hexianWeb/Third-Person-MC](https://github.com/hexianWeb/Third-Person-MC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

---
trigger: always_on
description: Guidelines for agentic coding assistants working on the dna-builder codebase.
---

# AGENTS.md

Guidelines for agentic coding assistants working on the dna-builder codebase.

## Build / Lint / Test Commands

### Frontend (Vue + TypeScript)

```bash
pnpm dev                              # Development server (http://localhost:1420/)
pnpm build                            # Production build
pnpm lint                             # Biome lint --fix + vue-tsc type checking
pnpm test                             # Run all tests (vitest run)
pnpm test src/data/tests/foo.test.ts  # Run single test file
pnpm coverage                         # Tests with coverage
pnpm format                           # Format with Biome
```

### Desktop App (Tauri + Rust)

```bash
pnpm tauri dev     # Development mode
pnpm tauri build   # Build desktop app
```

### Server (Bun + Elysia)

```bash
cd server && bun run dev   # Start dev server (port 8887)
bun run gen                # Generate database migrations
bun run migrate            # Run migrations
```

### Rust (MCP Server)

```bash
cd mcp_server && cargo build --release
```

## Code Style Guidelines

### Formatting (Biome)

- **Formatter**: Biome (`biome.json`) — NOT Prettier
- **Indentation**: 4 spaces (no tabs)
- **Line width**: 140 characters
- **Semicolons**: `asNeeded` (omit when possible)
- **Quotes**: Double quotes
- **Trailing commas**: ES5 style
- **Arrow parens**: As needed (omit for single param)
- **Line endings**: LF
- **Imports**: Auto-organized by Biome assist

### TypeScript Config

- **Strict mode**: Enabled (`strict: true`)
- **No unused locals/params**: Enforced
- **Path alias**: `@/*` → `./src/*`
- **Target**: ESNext, bundler module resolution
- **JSX**: Preserve with Vue JSX import source
- **Decorators**: `experimentalDecorators` enabled

### Vue / TypeScript Conventions

**Imports**: Named imports, type-only imports with `type` keyword

```typescript
import { ref, computed } from "vue"
import { defineStore } from "pinia"
import type { SomeType } from "./types"
```

**Naming**:

- Components: PascalCase (`Icon.vue`, `CharBuildView.vue`)
- Utilities/Composables: camelCase (`useCharSettings`, `formatProp`)
- Stores: `use` prefix (`useGameStore`, `useUIStore`)
- Constants: UPPER_SNAKE_CASE (`GAME_PROCESS`)
- Types/Interfaces: PascalCase (`CharBuild`, `LeveledWeapon`)

**Vue Specifics**:

- `<script setup lang="ts">` for all components
- Composition API only (no Options API)
- Pinia with `defineStore()` for state
- `defineProps<>()` and `defineEmits<>()` for type-safe props/emits
- Auto component imports via `unplugin-vue-components/vite`

**Error Handling**:

```typescript
async function someAsync() {
    try {
        const result = await apiCall()
        return result
    } catch (error) {
        console.error("Operation failed", error)
        return null
    }
}
```

### Rust (Tauri / MCP Server)

- Functions: snake_case, Structs: PascalCase, Constants: UPPER_SNAKE_CASE
- Tauri commands: `#[tauri::command]`, return `Result<T, String>`
- Error handling: `Result<T, E>` with `?` operator

## Testing Guidelines

- **Framework**: Vitest
- **Location**: `src/data/tests/` or alongside source files
- **Naming**: `*.test.ts`
- **Excludes**: `server/**`, `externals/**`
- **Coverage thresholds**: Lines 80%, Functions 80%, Branches 70%, Statements 80%

```typescript
import { describe, it, expect } from "vitest"

describe("Feature", () => {
    it("should do something", () => {
        expect(functionUnderTest()).toBe(expected)
    })
})
```

## Project Structure

```
src/
├── components/      # Vue components
├── data/            # Game data and calculations (tests in data/tests/)
├── store/           # Pinia stores
├── views/           # Page components
├── api/             # API calls
├── utils/           # Utility functions
└── router.ts        # Route config
server/              # Bun + Elysia backend
src-tauri/           # Tauri Rust backend
mcp_server/          # MCP server (Rust)
tools/               # Dev tools (i18n_tool.ts, icon_tool.ts)
externals/dna-api/   # DNA API package
public/i18n/         # Translation files
```

## Key Technologies

- **Frontend**: Vue 3, TypeScript, Tailwind CSS v4, daisyUI v5, reka-ui
- **State**: Pinia v3
- **Routing**: Vue Router
- **i18n**: i18next
- **Desktop**: Tauri 2 (Rust + WebView2)
- **Server**: Bun + Elysia + Drizzle ORM + GraphQL (graphql-yoga)
- **Testing**: Vitest
- **Build**: Vite v7
- **Linting/Formatting**: Biome

## Dev Tools

- **i18n**: `bun tools/i18n_tool.ts export|import` — export missing translations, import completed ones
- **Icons**: `bun tools/icon_tool.ts add|check|clean|list` — manage icons in `src/components/Icon.vue` use when lint error on icon not found
- **API gen**: `pnpm gen` — generate API calls from tools/generateApiCalls.ts

## Git Hooks (Husky)

Pre-commit hook auto-runs: version bump → `biome format` → `git add .`

**Warning**: The pre-commit hook runs `git add .` — all files in working directory get staged.

## Important Rules

1. **DO NOT RUN `pnpm dev` or `pnpm build`** — view http://localhost:1420/ directly in browser
2. **CHINESE COMMENTS**: Required for every function and complex logic block (JSDoc format)
3. **JSDoc**: Use for function documentation including params, return values, exceptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pa001024/dna-builder](https://github.com/pa001024/dna-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

---
trigger: always_on
description: InBrowser.app is a comprehensive Vue 3 + TypeScript monorepo containing 74+ workspace projects that provides browser-based utility tools. All tools run entirely client-side for privacy, processing data locally without server communication.
---

# Copilot Instructions for InBrowser.app

## Repository Overview

InBrowser.app is a comprehensive Vue 3 + TypeScript monorepo containing 74+ workspace projects that provides browser-based utility tools. All tools run entirely client-side for privacy, processing data locally without server communication.

**Key Technologies:**
- Vue 3 with Composition API (`<script setup>`)
- TypeScript for type safety
- pnpm workspaces for monorepo management
- Vite for build tooling
- Naive UI component library
- Vue i18n for internationalization (25+ languages)
- PWA capabilities with Workbox

**Repository Size & Structure:**
- ~74 workspace packages across apps/, tools/, shared/, utils/, registry/
- Main application: `apps/web` (Vue 3 + Vite)
- Node.js >= 18.0.0, pnpm >= 8.0.0 required
- Uses pnpm catalog for centralized dependency management

## Build & Development Commands

**CRITICAL: Always run commands in sequence as specified. Commands must be run from repository root.**

### Bootstrap & Development
```bash
# Always start with this - installs all dependencies
pnpm install

# Start development server (runs main web app on localhost:5173)
pnpm dev
```

### Build & Validation Pipeline (run in this exact order)
```bash
# 1. Lint check (oxlint + eslint)
pnpm lint-check

# 2. Format check (Prettier)
pnpm format-check

# 3. TypeScript type checking (takes ~30-60 seconds)
pnpm type-check

# 4. Production build (takes ~30-45 seconds, will show expected warnings)
pnpm build
```

**Build Warnings (Expected & Safe to Ignore):**
- Large chunk warnings (>500kB) for svgo-node and index files
- Vue compilation warnings for QR code generator reactive assignments  
- PWA service worker generation messages

### Fix Commands (when validation fails)
```bash
# Auto-fix linting issues
pnpm lint

# Auto-format code
pnpm format
```

### Testing Setup (requires additional setup)
```bash
# Unit tests (currently no unit tests exist - vitest configured but no test files)
pnpm test:unit

# E2E tests (requires Playwright browser installation first)
pnpm exec playwright install  # Must run before e2e tests
pnpm test:e2e
```

**Important Notes:**
- Build shows expected warnings: large chunks (>500kB), Vue compilation issues, PWA generation
- Type checking takes significant time due to project size (~30-60 seconds)
- Tests require additional setup (Playwright browser installation for e2e)
- Clean generated test files if linting fails: `rm -rf apps/web/playwright-report apps/web/test-results`
- Always run `pnpm install` before any build commands

## Project Architecture

### Monorepo Structure
```
inbrowser.app/
├── apps/web/           # Main Vue 3 application (entry point)
├── tools/              # Tool implementations with UI (77+ tools)
│   ├── code/          # Code-related tools (formatters, converters)
│   ├── document/      # Document processors
│   ├── image/         # Image manipulation tools  
│   ├── network/       # Network utilities
│   └── text/          # Text processing tools
├── shared/            # Shared packages
│   ├── locale/        # i18n languages (@shared/locale)
│   ├── tools/         # Tool interfaces (@shared/tools)
│   └── ui/            # UI components (@shared/ui)
├── utils/             # Pure TypeScript utilities (no UI)
├── registry/          # Aggregates all tools and routes
└── [config files]
```

### Key Configuration Files
- `package.json` - Root package with all build scripts
- `pnpm-workspace.yaml` - Workspace configuration with catalog
- `eslint.config.ts` - ESLint configuration (Vue + TypeScript)
- `.prettierrc.json` - Prettier config (semi: false, singleQuote: true)
- `tsconfig.json` - TypeScript project references
- `apps/web/vite.config.ts` - Vite build configuration
- `apps/web/vitest.config.ts` - Vitest test configuration
- `apps/web/playwright.config.ts` - Playwright e2e test configuration

## Tool Development Patterns

First, read some tools as example.

### Creating New Tools (follow existing patterns exactly)
1. **Directory structure:** `tools/<domain>/<tool-slug>/src/`
2. **Required files:**
   - `info.ts` - Tool metadata with i18n
   - `routes.ts` - Vue Router routes
   - `<ToolName>View.vue` - Main component
   - `components/**` - To store components, you need organize the components well. Don't let a component code too long.
   - `index.ts` - Exports toolInfo
   - `package.json` - Dependencies

3. **Register in registry:**
```bash
# Add tool to registry workspace
# @tools/<tool-name> is defined in tool's package.json name
pnpm -F @registry/tools add --workspace @tools/<tool-name>
```

4. **Tool metadata pattern (info.ts):**
```typescript
export const toolInfo: ToolInfo = {
  toolID: 'unique-tool-id',
  path: '/tools/tool-slug',  // Must match routes.ts
  tags: ['category1', 'category2'],
  features: ['offline'],  // For client-side tools
  meta: {
    // All 25 supported languages required
    en: { name: 'Tool Name', description: '...' },
    zh: { name: '工具名称', description: '...' },
    // ... all other languages
  }
}
```

### Routing & Registration
- App routes composed in `apps/web/src/router/routes.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/InBrowserApp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vectify is a CLI tool that generates framework-specific icon components from SVG files. It supports 10+ frameworks (React, Vue, Svelte, Solid.js, Preact, Angular, Lit, Qwik, Astro, Vanilla JS) with full TypeScript support and automatic SVG optimization.

**Key characteristics:**
- Package manager: `pnpm@9.15.9`
- Node requirement: `>=18.0.0`
- Built with `tsup` (outputs both CJS and ESM)
- Template-based code generation using Handlebars

## Development Commands

### Build
```bash
pnpm build          # Build for production (CJS + ESM + types)
pnpm dev            # Build in watch mode
pnpm clean          # Remove dist directory
```

### Testing
```bash
pnpm test           # Run tests once
pnpm test:watch     # Run tests in watch mode
```

### Linting
```bash
pnpm lint           # Check for lint errors
pnpm lint:fix       # Fix lint errors automatically
```

### Testing Generated Icons (Manual)
```bash
# After making changes, test icon generation:
pnpm build
cd /tmp && mkdir test-vectify && cd test-vectify
npm init -y
npm install /path/to/vectify/dist

# Create test SVG files
mkdir icons
echo '<svg viewBox="0 0 24 24"><path d="M12 2L2 7v10l10 5 10-5V7z"/></svg>' > icons/test.svg

# Initialize and generate
npx vectify init    # Select framework, configure paths
npx vectify generate
```

## Architecture Overview

### Core Data Flow

The generation pipeline follows this flow:

```
SVG File → beforeParse Hook → SVGO Optimization → SVG Parser (Cheerio)
→ IconNode Array → Framework Strategy → Template Rendering (Handlebars)
→ afterGenerate Hook → File Writer → Formatter → onComplete Hook
```

### Key Architectural Patterns

#### 1. **IconNode Format** (Core Data Structure)

All SVG elements are converted to `IconNode` tuples for framework-agnostic representation:

```typescript
type IconNode = [
  SVGElementType, // 'path' | 'circle' | 'rect' | 'g' etc.
  Record<string, string | number>, // Attributes (camelCase)
  IconNode[]? // Optional children
]
```

Example:
```typescript
// SVG: <circle cx="12" cy="12" r="10" stroke-width="2"/>
// IconNode: ['circle', { cx: 12, cy: 12, r: 10, strokeWidth: 2 }]
```

This format is:
- Serializable to component code
- Framework-agnostic
- Compact for embedding in generated files

#### 2. **Strategy Pattern** (Framework Abstraction)

`src/generators/framework-strategy.ts` defines the `FrameworkStrategy` interface that all framework generators implement:

```typescript
interface FrameworkStrategy {
  name: Framework
  getComponentExtension: (typescript: boolean) => string
  getIndexExtension: (typescript: boolean) => string
  generateComponent: (name, iconNode, typescript, keepColors) => string
  generateBaseComponent: (typescript) => { code: string, fileName: string }
}
```

**Why this matters:**
- Adding a new framework requires implementing this interface
- Each framework has different base components:
  - React: `createIcon.tsx` (factory function)
  - Vue: `Icon.vue` (base component)
  - Svelte: `Icon.svelte` (base component)
  - Angular: `icon.component.ts` (base component class)

All strategies are registered in `FrameworkRegistry` singleton at `src/generators/framework-strategy.ts:339-393`.

#### 3. **Template Engine** (Code Generation)

Handlebars templates (`src/generators/templates/**/*.hbs`) separate logic from presentation.

**Important build detail:** Templates are NOT bundled by tsup. They're copied to `dist/templates/` via `scripts/copy-templates.js` (triggered by tsup's `onSuccess` hook in `tsup.config.ts:11`).

**Template path resolution** (`src/generators/templates/template-engine.ts:23-33`):
- CJS: Uses `__dirname` → `dist/templates/`
- ESM: Uses `import.meta.url` → `dist/templates/`

**Template structure:**
```
templates/
├── react/
│   ├── component.tsx.hbs      # Individual icon component
│   └── createIcon.tsx.hbs     # Base helper function
├── vue/
│   ├── component.ts.vue.hbs
│   └── icon.ts.vue.hbs        # Base Icon.vue component
├── svelte/
├── solid/
├── preact/
├── qwik/
├── angular/
├── lit/
├── astro/
└── vanilla/
```

Each template receives:
- `typescript`: boolean
- `componentName`: string (PascalCase)
- `formattedNodes`: string (IconNode array as code string)
- `keepColors`: boolean (preserve SVG colors vs. currentColor)

#### 4. **Config Loading with `jiti`**

`src/config/loader.ts` uses `jiti` to dynamically load TypeScript config files at runtime (no transpilation step needed). This allows users to write `vectify.config.ts` directly.

**Search order:**
1. `--config` CLI flag path
2. `vectify.config.ts`
3. `vectify.config.js`

**Path resolution:** All `input`/`output` paths are resolved relative to `configDir` (defaults to config file location).

### Critical Files by Function

| Function | File Path | Key Responsibilities |
|----------|-----------|---------------------|
| CLI Entry | `src/cli.ts` | Commander.js setup, command registration |
| Type Definitions | `src/types.ts` | `IconForgeConfig`, `IconNode`, `Framework` types |
| Main Orchestrator | `src/generators/index.ts` | Pipeline coordination, file writing, hooks execution |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hixb/vectify](https://github.com/hixb/vectify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->

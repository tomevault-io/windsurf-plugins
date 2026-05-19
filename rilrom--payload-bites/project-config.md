---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Payload bites is a monorepo containing bite-sized Payload v3 plugins and tools. Each plugin is published as an independent npm package under `@payload-bites/*`.

**Current plugins:**
- `image-search` - Search for images using providers (Unsplash, Pexels, Pixabay)
- `fullscreen-editor` - Fullscreen mode for Lexical rich text editor
- `audit-fields` - Add createdBy/lastModifiedBy audit fields
- `soft-delete` - Soft delete functionality for documents
- `activity-log` - Track CMS activity
- `broken-link-checker` - Detect and prevent broken links
- `content-freeze` - Freeze content during critical moments
- `astro-richtext-renderer` - Render Payload CMS Lexical richtext content in Astro

## Monorepo Structure

This is a pnpm workspace managed by Turbo:

```
payload-bites/
├── packages/           # Published plugins (source code)
│   ├── image-search/
│   ├── soft-delete/
│   ├── astro-richtext-renderer/
│   └── ...
├── apps/              # Demo/test apps for each plugin
│   ├── image-search/
│   ├── soft-delete/
│   ├── astro-richtext-renderer/
│   │   ├── payload/   # Payload CMS app
│   │   └── website/   # Astro website
│   └── ...
└── packages/
		├── typescript-config/    # Shared TypeScript configuration
    └── playwright-utilities/ # Shared Playwright helper functions
```

## Development Commands

### Root-level commands (uses Turbo)
- `pnpm dev` - Run dev mode across all packages
- `pnpm build` - Build all packages
- `pnpm lint` - Lint all packages (Biome + Stylelint)
- `pnpm tsc` - Type check all packages
- `pnpm test` - Run all tests (unit + E2E)
- `pnpm clean` - Clean build artifacts (node_modules, .next, .astro, .turbo, dist)

### Package-specific development
Use convenience scripts to develop specific packages:

```bash
# Develop a specific package (runs both the plugin and its demo app)
pnpm dev:image-search
pnpm dev:soft-delete
pnpm dev:activity-log
pnpm dev:audit-fields
pnpm dev:broken-link-checker
pnpm dev:content-freeze
pnpm dev:fullscreen-editor
pnpm dev:astro-richtext-renderer
```

Alternatively, use Turbo filters directly:

```bash
pnpm dev --filter PACKAGE_NAME --filter @payload-bites/PACKAGE_NAME
```

### Individual package scripts
Within `packages/*/`:
- `pnpm dev` - Build in watch mode (TypeScript + SWC)
- `pnpm build` - Production build
- `pnpm lint` - Run Biome linter (and Stylelint for packages with SCSS)
- `pnpm tsc` - Type check without emit
- `pnpm test` - Run all tests (unit + E2E)
- `pnpm test:unit` - Run unit tests only (Vitest)
- `pnpm test:e2e` - Run E2E tests only (Playwright)

Within `apps/*/` (Next.js apps):
- `pnpm dev` - Start Next.js dev server with Turbopack
- `pnpm build` - Build Next.js app
- `pnpm generate:types` - Generate Payload types
- `pnpm generate:importmap` - Generate Payload import map

Within Astro apps (e.g., `apps/astro-richtext-renderer/website/`):
- `pnpm dev` - Start Astro dev server
- `pnpm build` - Build Astro site
- `pnpm preview` - Preview production build

## Plugin Architecture

### Standard plugin structure
```
packages/PLUGIN_NAME/
├── src/
│   ├── index.ts              # Main plugin export
│   ├── types.ts              # TypeScript types
│   ├── defaults.ts           # Default plugin options
│   ├── translations.ts       # i18n translations
│   ├── components/           # React components
│   ├── endpoints/            # API endpoints
│   ├── classes/              # Core logic classes
│   ├── utils/                # Utility functions
│   └── exports/
│       ├── client.ts         # Client-side component exports
│       ├── rsc.ts            # Server component exports
│       └── utilities.ts      # Utility exports
├── package.json
├── .swcrc                    # SWC build configuration
└── README.md
```

### Plugin implementation pattern
All plugins follow this functional pattern:

```typescript
export const pluginNamePlugin =
	(pluginOptions?: PluginOptions) =>
	(incomingConfig: Config): Config => {
		// Merge options with defaults
		const mergedOptions = Object.assign(defaultOptions, pluginOptions);

		const config = { ...incomingConfig };

		// Check if plugin is enabled
		if (mergedOptions.enabled === false) {
			return config;
		}

		// Add translations
		config.i18n = {
			translations: deepMerge(translations, config.i18n?.translations),
		};

		// Modify collections/globals/endpoints as needed
		// ...

		return config;
	};
```

### Component exports
Plugins export components via separate entry points defined in `package.json`:

```json
{
	"exports": {
		".": "./dist/index.js",
		"./client": "./dist/exports/client.js",
		"./rsc": "./dist/exports/rsc.js"
	}
}
```

Components are injected into Payload config using string paths:
```typescript
"@payload-bites/plugin-name/client#ComponentName"
```

## Build System

### TypeScript + SWC
Packages use dual compilation:
- **TypeScript** (`tsc`) - Generates type definitions (`.d.ts`)
- **SWC** - Transpiles code for fast builds

Build process copies non-TS assets (CSS, images, etc.) via `copyfiles`.

### TypeScript configuration
All packages extend `@payload-bites/typescript-config/base.json`:
- Target: ES2022

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rilrom/payload-bites](https://github.com/rilrom/payload-bites) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

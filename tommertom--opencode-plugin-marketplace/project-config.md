---
trigger: always_on
description: This guide helps coding agents understand the codebase structure, build processes, and coding standards.
---

# Agent Guide for OpenCode Plugin Marketplace

This guide helps coding agents understand the codebase structure, build processes, and coding standards.

## Project Overview

Community-driven marketplace for OpenCode plugins built with:
- **Frontend**: SolidJS + TypeScript + Vite
- **Validation**: Node.js scripts with Ajv JSON Schema validator
- **Hosting**: Firebase Hosting
- **Structure**: Monorepo with `/web` (frontend) and `/scripts` (validation)

## Build & Development Commands

### Web Application (Frontend)
```bash
cd web
npm install                # Install dependencies
npm run dev               # Start dev server (Vite)
npm run build             # Production build
npm run preview           # Preview production build
```

### Validation Scripts
```bash
cd scripts
npm install               # Install dependencies
npm run validate          # Validate all plugin JSON files
```

### Running Single Validation
To validate a specific plugin file:
```bash
cd scripts
node validate-plugins.js  # Validates all plugins in /plugins directory
```
**Note**: There's no built-in single-file validation. The script validates all `.plugin.json` files.

## Testing

**Current Status**: No test suite exists. The project relies on:
- JSON schema validation via `scripts/validate-plugins.js`
- TypeScript type checking via `tsc --noEmit`
- CI validation via GitHub Actions

## Code Style Guidelines

### TypeScript Configuration
- **Target**: ES2020
- **Strict Mode**: Enabled (`strict: true`)
- **Unused Variables**: Error (`noUnusedLocals`, `noUnusedParameters`)
- **Module System**: ESNext with bundler resolution
- **JSX**: Preserve with `solid-js` import source

### Import Conventions
```typescript
// External dependencies first
import { createSignal, For, Show } from 'solid-js';
import { SolidMarkdown } from 'solid-markdown';

// Type imports separately
import type { Plugin } from './data/types';
import type { Component } from 'solid-js';

// Local modules last
import './App.css';
```

### Naming Conventions
- **Files**: PascalCase for components (`PluginCard.tsx`), camelCase for utilities (`github.ts`)
- **Components**: PascalCase function declarations (`export function PluginCard()`)
- **Interfaces**: PascalCase with descriptive names (`PluginCardProps`, `CacheEntry`)
- **Variables**: camelCase (`selectedPlugin`, `fetchGitHubStars`)
- **Constants**: SCREAMING_SNAKE_CASE for true constants (`CACHE_KEY`, `CACHE_DURATION`)
- **Plugin Files**: kebab-case with `.plugin.json` suffix (`opencode-example.plugin.json`)

### Type Definitions
- Use `interface` for object shapes and props
- Use `type` for unions, intersections, or complex types
- Always type component props explicitly
- Prefer explicit return types for exported functions
- Use optional chaining (`?.`) and nullish coalescing (`??`) when appropriate

### SolidJS Patterns
```typescript
// Use createSignal for reactive state
const [selectedPlugin, setSelectedPlugin] = createSignal<Plugin | null>(null);

// Use Show for conditional rendering
<Show when={condition} fallback={<div>Loading...</div>}>
  <Content />
</Show>

// Use For for lists
<For each={items}>
  {(item) => <Item data={item} />}
</For>

// Use onMount for lifecycle
onMount(async () => {
  // Initialization code
});
```

### Error Handling
```typescript
// Silent failures with null returns for non-critical operations
try {
  const data = await fetch(url);
  return data;
} catch {
  return null;  // No console.error, just graceful degradation
}

// Empty catch blocks for storage errors
try {
  localStorage.setItem(key, value);
} catch {
  // Ignore storage errors
}
```

### Formatting Standards
- **Indentation**: 2 spaces (no tabs)
- **Quotes**: Single quotes for strings, double for JSX attributes
- **Semicolons**: Required
- **Line Length**: ~100-120 characters (not strictly enforced)
- **Trailing Commas**: Not used in objects/arrays

### JSON Schema Files
- Plugin files in `/plugins` must follow `schema/plugin.schema.json`
- Filename must match the `name` field: `<name>.plugin.json`
- Use lowercase, alphanumeric characters, and hyphens only
- All dates in `YYYY-MM-DD` format
- All URLs must be valid URIs

### Comments
- Use comments sparingly; prefer self-documenting code
- Use JSDoc for exported utility functions when behavior is non-obvious
- No inline comments in simple component code

## File Structure

```
/plugins/                   # Plugin JSON files
/schema/                    # JSON schema definitions
/scripts/                   # Validation scripts (Node.js)
  validate-plugins.js       # Main validation script
/web/                       # Frontend application
  /src/
    /components/            # SolidJS components
    /data/                  # Data loading and types
    /utils/                 # Utility functions
    App.tsx                 # Main app component
    index.tsx               # Entry point
```

## Common Patterns

### Adding a New Plugin
1. Create `<plugin-name>.plugin.json` in `/plugins`
2. Ensure `name` field matches filename
3. Run `cd scripts && npm run validate`
4. Submit PR (CI will validate automatically)

### Modifying Components
1. Edit component in `/web/src/components/`
2. Maintain existing prop interface patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tommertom/opencode-plugin-marketplace](https://github.com/Tommertom/opencode-plugin-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

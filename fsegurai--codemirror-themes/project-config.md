---
trigger: always_on
description: **codemirror-themes** is a **monorepo of 26 CodeMirror 6 theme packages**. The architecture separates individual themes into independent npm packages under `packages/`, with a unified build system and demo application. Each theme is independently publishable on npm under the `@fsegurai/codemirror-theme-*` namespace.
---

# AGENTS.md - AI Agent Guide for codemirror-themes

## Project Overview

**codemirror-themes** is a **monorepo of 26 CodeMirror 6 theme packages**. The architecture separates individual themes into independent npm packages under `packages/`, with a unified build system and demo application. Each theme is independently publishable on npm under the `@fsegurai/codemirror-theme-*` namespace.

### Architecture & Data Flow

```
Root (workspace config, shared tsconfig, build tools)
├── packages/[theme-name]/          ← Individual theme packages (26 total)
│   ├── src/index.ts               ← Theme definition (exports theme + mergeStyles)
│   ├── src/utils.ts               ← Shared styling helpers (colors, layouts)
│   └── package.json               ← Scoped package (@fsegurai/codemirror-theme-*)
├── packages/bundle/               ← Re-exports all themes for convenience
├── demo/                          ← Development & showcase application
│   ├── scripts/index.ts           ← README renderer & demo UI
│   ├── scripts/playground.ts      ← Interactive theme switcher
│   └── styles/themes.css          ← Live theme preview styling
└── build system (vite, typescript, eslint)
```

**Key Pattern**: Each theme is an **independent, publishable npm package** that exports:
- `export const [themeName]` - CodeMirror Extension
- `export const [themeName]MergeStyles` - Merge view styles (IMergeRevertStyles object)

## Critical Developer Workflows

### Build & Publish Pipeline
```bash
bun run build:packages     # Compiles all packages via cm-buildhelper (not vite)
bun run start              # Dev server with live reload (vite)
bun test                   # Runs tests across all packages
bun run lint               # ESLint + TypeScript checking
```

**Important**: Theme packages use **cm-buildhelper** (a CodeMirror-specific build tool), NOT rollup. This happens automatically via the `prepare` script in each package.json.

### Theme Development Workflow
When modifying a theme (e.g., `packages/gruvbox-dark/src/index.ts`):
1. Edit theme colors and highlight rules
2. Run `bun run build:packages` to trigger cm-buildhelper
3. Start demo with `bun run start` to see changes in real-time
4. The playground.ts automatically detects the built exports

### Adding a New Theme
1. Create `/packages/new-theme/src/index.ts` with CodeMirror Extension structure
2. Copy structure from existing theme (e.g., abcdef)
3. Follow the pattern: define color palette → create HighlightStyle → create EditorView extension
4. Add `utils.ts` with IMergeRevertStyles for merge view support
5. Create `package.json` (copy existing, update name/version)
6. Build with `bun run build:packages`
7. Add export to `packages/bundle/src/index.ts`

## Project-Specific Patterns & Conventions

### Theme Definition Pattern
Every theme exports exactly this structure:
```typescript
// Core extension
export const themeName: Extension = EditorView.theme({ ... }, { dark: true/false })

// Syntax highlighting
export const themeNameHighlight = syntaxHighlighting(HighlightStyle.define([...]))

// Merge/diff view support
export const themeNameMergeStyles: IMergeRevertStyles = {
  backgroundColor, borderColor, buttonColor, buttonHoverColor
}
```

### Color Palette Convention
Themes use a **base00-0F naming scheme** (inherited from Base16):
- `base00`: Background
- `base01-07`: UI elements (selection, gutters, panels)
- `base08-0F`: Syntax highlighting (keywords, comments, etc.)
- Additional: `invalid`, `darkBackground`, `cursor`, `selection`

See `packages/abcdef/src/index.ts` lines 1-35 as reference.

### Styling Utilities (utils.ts)
All themes share **standardized UI element styling** in utils.ts:
- `generalContent`: Font family (JetBrains Mono, Consolas), size, line-height
- `generalGutter`: Padding, font size, line height
- `generalDiff`: Diff/merge visual rules (strikethrough for deletions, etc.)
- `generalCursor`, `generalPanel`, `generalLine`, etc.

These are imported and used in index.ts to maintain consistency. **DO NOT change general* exports** without coordinating across all themes.

### ESLint Rules (Important!)
The project uses single quotes, 2-space indent, and enforces operator-linebreak='before':
```javascript
// ✓ Correct
const colors = {
  base00: '#0a0e14'
};

// ✗ Wrong
const colors = {
  base00: "#0a0e14"
};
```

See eslint.config.js for full rules. Run `bun run lint` before commits.

### TypeScript Configuration
- Target: ES2022
- Module: ES2022 (ESM only)
- Strict: true (no implicit any, etc.)
- Module resolution: Bundler (for monorepo)
- Exclude test files: `**/*.test-d.ts, **/*.spec.ts`

## Integration Points & Dependencies

### Key External Dependencies
- **@codemirror/view, @codemirror/state, @codemirror/language**: Core editor API
- **@lezer/highlight**: Syntax highlighting token definitions
- **@codemirror/merge**: Diff view (optional, used for mergeStyles exports)

### Cross-Component Communication

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fsegurai/codemirror-themes](https://github.com/fsegurai/codemirror-themes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

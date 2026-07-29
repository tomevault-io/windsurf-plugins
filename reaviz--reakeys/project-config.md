---
trigger: always_on
description: **reakeys** is a React hotkeys library that provides a `useHotkeys` hook for registering keyboard shortcuts in React applications. It wraps the `ctrl-keys` library and provides a simple, declarative API for managing keyboard shortcuts with support for global and element-scoped bindings.
---

# CLAUDE.md - AI Assistant Guide for reakeys

## Project Overview

**reakeys** is a React hotkeys library that provides a `useHotkeys` hook for registering keyboard shortcuts in React applications. It wraps the `ctrl-keys` library and provides a simple, declarative API for managing keyboard shortcuts with support for global and element-scoped bindings.

- **Version**: 2.0.6
- **License**: Apache-2.0
- **Package Type**: ES Module
- **React Compatibility**: React 16+

## Codebase Structure

```
reakeys/
├── src/                    # Source code
│   ├── index.ts           # Main exports
│   ├── useHotkeys.ts      # Core hook implementation
│   ├── utils.ts           # Utility functions (isMac, getHotkeyText, MODIFIER_KEY)
│   └── Hotkey.story.tsx   # Storybook stories for testing/demos
├── .storybook/            # Storybook configuration
│   ├── main.ts            # Storybook main config
│   ├── preview.tsx        # Storybook preview config
│   ├── theme.ts           # Storybook theme
│   └── manager.ts         # Storybook manager config
├── .github/               # GitHub configuration
│   └── workflows/         # CI/CD workflows
│       ├── build.yml      # PR build workflow
│       ├── release.yml    # Release workflow
│       └── npm-publish.yml # NPM publish workflow
├── dist/                  # Build output (generated)
├── vite.config.ts         # Vite build configuration
├── tsconfig.json          # TypeScript configuration
├── eslint.config.js       # ESLint flat config
├── .prettierrc            # Prettier configuration
└── package.json           # Project dependencies and scripts
```

## Key Source Files

### `src/useHotkeys.ts`
The main React hook that manages keyboard shortcuts. Key features:
- Registers global event listeners for `keypress`, `keyup`, and `keydown`
- Supports element-scoped shortcuts via `ref` property
- Automatically ignores shortcuts when typing in input fields
- Uses `ctrl-keys` library for key handling
- Manages a global `hotkeys` array for tracking all registered shortcuts

### `src/utils.ts`
Utility functions:
- `isMac()`: Detects macOS platform
- `getHotkeyText(hotkey)`: Formats hotkey string for display (e.g., `mod+shift+a` → `⌘+⇧+a`)
- `MODIFIER_KEY`: Platform-specific modifier key symbol (`⌘` on Mac, `CTRL` elsewhere)

### `src/index.ts`
Simple re-export of `useHotkeys` and `utils`.

## Development Commands

```bash
# Install dependencies
npm install

# Start Storybook dev server (port 9009)
npm start

# Build library for production
npm run build

# Build Storybook
npm run build-storybook

# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Lint code
npm run lint

# Fix lint issues
npm run lint:fix

# Format code with Prettier
npm run prettier

# Run Chromatic visual tests
npm run chromatic
```

## Architecture & Key Concepts

### HotkeyShortcuts Interface
```typescript
interface HotkeyShortcuts {
  name: string;                           // Required: Shortcut name
  keys: string | string[];                // Required: Key combo(s)
  callback: Callback;                     // Required: Handler function
  ref?: RefObject<HTMLElement | null>;    // Optional: Element scope
  disabled?: boolean;                     // Optional: Disable shortcut
  action?: 'keypress' | 'keydown' | 'keyup'; // Optional: Event type
  description?: string;                   // Optional: Description
  category?: string;                      // Optional: Category grouping
  hidden?: boolean;                       // Optional: Hide from listing
}
```

### Key Modifiers
- `mod` or `modifier`: Platform-specific (Meta on Mac, Ctrl elsewhere)
- `meta`: Meta key (Command on Mac)
- `ctrl`: Control key
- `shift`: Shift key
- `alt`: Alt key

### Global vs Element-Scoped Shortcuts
- **Global**: No `ref` property - listens on `window`
- **Element-scoped**: With `ref` property - listens on the referenced element

### Input Focus Handling
Shortcuts are automatically disabled when:
- User is typing in an `<input>` (except checkbox, radio, button types)
- User is typing in a `<textarea>`
- User is typing in a `<select>`
- Element has `contentEditable` enabled

## Code Style & Conventions

### TypeScript
- Target: ES2015
- Module: ESNext
- Strict mode enabled
- JSX: React

### ESLint
- ESLint 10 flat config (`eslint.config.js`)
- Composes: `@eslint/js` recommended, `typescript-eslint` (parser-only), `eslint-plugin-react`, `eslint-plugin-react-hooks`, `eslint-plugin-storybook`, `eslint-config-prettier`

### Prettier
- Semicolons: Yes
- Single quotes: Yes

### Pre-commit Hooks
Uses Husky + lint-staged to run Prettier on staged files before commit.

## Testing

- Test framework: Vitest
- Environment: jsdom
- Currently configured with `--passWithNoTests` flag
- Coverage provider: `@vitest/coverage-v8`
- Stories in `*.story.tsx` files serve as visual integration tests via Chromatic

## Build System

Uses Vite 8 with two modes:
1. **Library mode** (`npm run build`): Builds for npm distribution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reaviz/reakeys](https://github.com/reaviz/reakeys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

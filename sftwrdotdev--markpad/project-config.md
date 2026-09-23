---
trigger: always_on
description: This file contains guidelines for AI agents working on the Markpad codebase.
---

# AGENTS.md - Coding Guidelines for Markpad

This file contains guidelines for AI agents working on the Markpad codebase.

## Project Overview

Markpad is a Tauri v2 application with:
- **Frontend**: Svelte 5 + TypeScript + Vite
- **Backend**: Rust (Tauri commands)
- **Purpose**: Markdown viewer and text editor

## Build Commands

### Frontend (TypeScript/Svelte)
```bash
# Development
npm run dev              # Start Vite dev server
npm run dev:installer    # Run in installer mode

# Building
npm run build            # Build frontend for production
npm run preview          # Preview production build

# Type Checking
npm run check            # Run svelte-check (type checking)
npm run check:watch      # Watch mode type checking
```

### Backend (Rust)
```bash
cd src-tauri

cargo build              # Build Rust code
cargo build --release    # Release build

# Testing
cargo test               # Run all Rust tests
cargo test <test_name>   # Run specific test

# Linting/Formatting
cargo check              # Check for errors
cargo clippy             # Run clippy linter
cargo fmt                # Format code
```

### Tauri Commands
```bash
npm run tauri dev        # Run Tauri in dev mode
npm run tauri build      # Build Tauri application
```

## Code Style Guidelines

### TypeScript/Svelte

- **Indentation**: Use tabs (not spaces)
- **Quotes**: Single quotes for strings
- **Semicolons**: Optional but be consistent
- **Strict Mode**: Enabled - always define types explicitly

#### Imports
- Group imports: external libs first, then internal modules
- Use `.js` extension for relative imports (SvelteKit convention)
- Example:
```typescript
import { onMount } from 'svelte';
import { tabManager } from '../stores/tabs.svelte.js';
```

#### Svelte 5 Runes
Always use Svelte 5 runes for reactivity:
- `$state()` for reactive state
- `$derived()` for computed values
- `$effect()` for side effects
- `$props()` for component props with explicit types
- `$bindable()` for two-way binding

#### Naming Conventions
- Components: PascalCase (e.g., `Editor.svelte`)
- Stores: camelCase with `.svelte.ts` extension (e.g., `tabs.svelte.ts`)
- Functions/Variables: camelCase
- Types/Interfaces: PascalCase
- Event handlers: Prefix with `on` (e.g., `onsave`, `onnew`)

#### Props Pattern
Always type props explicitly using `$props()`:
```typescript
let {
    value = $bindable(),
    onsave,
    theme = 'system',
} = $props<{
    value: string;
    onsave?: () => void;
    theme?: 'system' | 'light' | 'dark';
}>();
```

### Rust

- **Indentation**: 4 spaces
- **Error Handling**: Use `Result<T, String>` for Tauri commands, propagate with `?`
- **Naming**: snake_case for functions/variables, PascalCase for types

#### Tauri Commands
Always mark with `#[tauri::command]` and return `Result<T, String>`:
```rust
#[tauri::command]
fn read_file_content(path: String) -> Result<String, String> {
    fs::read_to_string(path).map_err(|e| e.to_string())
}
```

#### Platform-Specific Code
Use conditional compilation for platform-specific features:
```rust
#[cfg(target_os = "windows")]
// Windows-specific code

#[cfg(not(target_os = "windows"))]
// Fallback for other platforms
```

## Project Structure

```
src/
  lib/
    components/          # Svelte components
    stores/              # Svelte 5 stores (.svelte.ts)
  routes/                # SvelteKit routes
src-tauri/
  src/
    main.rs             # Entry point
    lib.rs              # Main library with Tauri commands
    setup.rs            # Installation/setup logic
```

## Key Patterns

### State Management
- Use Svelte 5 runes-based stores in `src/lib/stores/`
- Export singleton instances (e.g., `export const tabManager = new TabManager()`)

### Tauri Invoke
Use `invoke()` from `@tauri-apps/api/core` for Rust commands:
```typescript
import { invoke } from '@tauri-apps/api/core';
const result = await invoke<string>('command_name', { arg: value });
```

### File Operations
All file operations go through Rust commands - never use Node.js fs APIs.

## Testing

- **Rust**: `cargo test` in `src-tauri/` directory
- **Frontend**: two runners, split by glob so a file belongs to exactly one.
  - `npm test` runs `scripts/*.test.ts` (`node --test --import tsx`).
  - `npm run test:vitest` runs `scripts/*.spec.ts` (vitest, jsdom, Svelte plugin).
- **CI**: Runs `npm audit`, `npm run check`, `npm test`, `npm run test:vitest` and
  `cargo test` on PRs

Two kinds of assertion live in this suite: behavior tests, which import and run the real
modules, and source-shape assertions, which match the source text for a contract the
compiler cannot check — a Tauri command name, an i18n key, a second copy of a fixed
behavior. A green run is not a claim that every behavior is covered; a source-shape
assertion passes as long as the line it matches is still spelled that way.

### Which runner a test belongs to

**Write it as `*.spec.ts` (vitest) when the subject is a `.svelte.ts` runes module, or when
it needs a real DOM.** `node --test` cannot load either. A runes module reached from
`node --test` only runs because the test installs `$state`/`$derived`/`$effect` onto
`globalThis` itself, and a hand-written rune is not the compiler's: it misses `$derived`'s

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sftwrdotdev/Markpad](https://github.com/sftwrdotdev/Markpad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

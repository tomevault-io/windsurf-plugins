---
trigger: always_on
description: This document serves as a comprehensive guide to the DesQTA codebase, practices, style, and architecture. It is intended for AI agents and developers to understand the project structure and conventions.
---

# DesQTA Codebase Guide for Agents

This document serves as a comprehensive guide to the DesQTA codebase, practices, style, and architecture. It is intended for AI agents and developers to understand the project structure and conventions.

**For agents:** Before editing, check Section 11 (Project Skills) for a matching skill—read and follow it when applicable. Use the Svelte MCP server for Svelte development when available.

## 1. Tech Stack Overview

### Core Frameworks

- **Frontend Framework:** Svelte 5 (Runes mode enabled: `$state`, `$derived`, `$effect`)
- **Build Tool:** Vite 6
- **Language:** TypeScript 5.6+
- **CSS Framework:** Tailwind CSS 4.1.13 (using `@theme inline` and `oklch` colors)
- **Desktop/Mobile Engine:** Tauri 2.9.2 (Rust backend)
- **Database:** SQLite (via `rusqlite` in Rust) & IndexedDB (via `idb` in frontend for caching)

### Key Libraries

- **Icons:** `svelte-hero-icons`
- **Charts:** `layerchart` (implied usage), D3
- **Internationalization:** `svelte-i18n`
- **State Management:** Svelte 5 Runes (local/component) + Svelte Stores (global)
- **Editor:** Tiptap (headless wrapper for ProseMirror)
- **Motion/Animations:** `svelte/transition`, `svelte/easing`, `motion`

---

## 2. Project Structure

### Root Directory

- `src/`: Frontend source code (SvelteKit)
- `src-tauri/`: Backend source code (Rust/Tauri)
- `static/`: Static assets (images, themes, service worker)
- `docs/`: Documentation (Note: May be outdated, prioritize this file)

### Frontend (`src/`)

- **`routes/`**: SvelteKit routing.
  - `+layout.svelte`: Main application shell (Sidebar, Header, Auth check).
  - `+page.svelte`: Dashboard/Home page.
  - Directories represent routes (e.g., `courses/`, `assessments/`).
- **`lib/`**:
  - **`components/`**: Reusable UI components (PascalCase).
  - **`services/`**: Singleton service classes for business logic (e.g., `authService.ts`, `weatherService.ts`).
  - **`stores/`**: Global Svelte stores (e.g., `theme.ts`, `themeBuilderSidebar.ts`).
  - **`utils/`**: Helper functions (e.g., `logger.ts`, `netUtil.ts`).
  - **`i18n/`**: Localization configuration.
- **`app.css`**: Global styles and Tailwind v4 theme configuration.

### Backend (`src-tauri/src/`)

- **`main.rs`**: Entry point, command registration, plugin setup.
- **`lib.rs`**: Library entry point (often used by `main.rs`).
- **`services/`**: Backend business logic modules.
- **`utils/`**: Helper modules (filesystem, database, logging).
- **`auth/`**: Authentication logic.

---

## 3. Code Style & Practices

### Frontend (Svelte/TypeScript)

- **Svelte 5 Runes:** MUST use Svelte 5 Runes for reactivity.
  - State: `let count = $state(0);`
  - Derived: `let double = $derived(count * 2);`
  - Side Effects: `$effect(() => { ... });`
  - Props: `let { prop1, prop2 }: Props = $props();`
- **Components:** PascalCase filenames.
- **Imports:** Use `$lib/` alias for accessing `src/lib`.
- **Services:** Encapsulate logic in services (singleton objects) rather than placing complex logic in components.
- **Async/Await:** Prefer `async/await` over raw Promises.
- **Logging:** Use `logger` utility (`logger.info`, `logger.error`) instead of `console.log` for structured logging.
- **Type Safety:** Strict TypeScript usage. Define interfaces for props and data structures.

### Backend (Rust)

- **Commands:** Logic exposed to frontend via `#[tauri::command]`.
- **Modularity:** Split logic into modules (`mod`) under `src-tauri/src/`.
- **Error Handling:** Return `Result<T, String>` (or specialized error types) to frontend.
- **Async:** Use `tokio` runtime features for async operations.
- **Naming:** snake_case for functions, modules, and variables.

### UI Styling (Tailwind CSS v4)

- **Configuration:** Theme defined in `src/app.css` using `@theme inline`.
- **Colors:** Use CSS variables with `oklch` color space (e.g., `--accent`, `--background`).
- **Dark Mode:** Support `dark:` variants. Theming handles class switching on `html` element.
- **Classes:** Mobile-first approach (`sm:`, `lg:`).
- **Custom Variants:** `@custom-variant dark (&:is(.dark *));` defined in CSS.

---

## 4. Naming Conventions

| Entity                | Convention            | Example                                 |
| :-------------------- | :-------------------- | :-------------------------------------- |
| **Svelte Components** | PascalCase            | `AppHeader.svelte`, `TodoList.svelte`   |
| **TS/JS Files**       | camelCase             | `authService.ts`, `utils.ts`            |
| **Svelte Stores**     | camelCase             | `themeStore`, `accentColor`             |
| **Rust Files**        | snake_case            | `seqta_config.rs`, `main.rs`            |
| **Rust Functions**    | snake_case            | `save_settings`, `check_session_exists` |
| **CSS Classes**       | kebab-case (Tailwind) | `flex-col`, `text-zinc-900`             |
| **Directories**       | kebab-case (mostly)   | `user-documentation`, `src-tauri`       |

---

## 5. Architecture & Data Flow

1.  **Initialization:**
    - `src/lib/services/startupService.ts` handles initial data loading from SQLite/IndexedDB for "instant" UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BetterSEQTA/DesQTA](https://github.com/BetterSEQTA/DesQTA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

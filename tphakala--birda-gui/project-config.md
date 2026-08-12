---
trigger: always_on
description: Desktop GUI for the **birda** bird species detection CLI. Built with Electron + Svelte 5 + TypeScript.
---

# Birda GUI - LLM Context

Desktop GUI for the **birda** bird species detection CLI. Built with Electron + Svelte 5 + TypeScript.

## 🔍 Understanding the Codebase — USE LEANN FIRST

**CRITICAL:** Before reading files, grepping, or exploring the codebase to understand how something works, you MUST use LEANN first. LEANN is a semantic vector search system that provides accurate, context-aware answers about the codebase architecture and implementation patterns.

### When You Need to Understand Code or Project Structure

1. **ALWAYS START WITH LEANN** — Use `fish -c "leann ask birda-gui '<your question>'"` for:
   - "How does [feature/system] work?"
   - "Where is [functionality] implemented?"
   - "What patterns are used for [architecture concern]?"
   - "How do [components] interact?"

2. **THEN use direct tools** (Read/Grep/Glob) only for:
   - Reading specific files LEANN pointed you to
   - Examining exact line-level implementation details
   - Verifying or modifying code you already understand

**Example workflow:**

❌ **WRONG:** Grep for "IPC" → Read multiple files → Try to piece together understanding

✅ **CORRECT:** `leann ask birda-gui "How does the Electron IPC architecture work?"` → Get comprehensive answer with file references → Read specific files if needed for implementation details

**LEANN provides:**

- Semantic understanding (not just keyword matching)
- Architectural context and patterns
- Accurate file and module references
- Explanation of how components interact
- AST-aware code context

See the [LEANN section](#leann-low-storage-vector-index) below for detailed commands and usage patterns.

## Tech Stack

| Layer        | Technology                       | Version                                |
| ------------ | -------------------------------- | -------------------------------------- |
| Runtime      | Electron                         | 40.x                                   |
| UI Framework | Svelte                           | 5.x (runes API, **not** legacy stores) |
| CSS          | Tailwind CSS v4 + daisyUI v5     | 4.1.x / 5.5.x                          |
| Language     | TypeScript                       | 5.9.x (strict mode)                    |
| Bundler      | Vite via electron-vite           | 7.x / 5.x                              |
| Database     | better-sqlite3                   | 12.x                                   |
| i18n         | Paraglide (compile-time)         | 2.x                                    |
| Maps         | MapLibre GL + svelte-maplibre-gl | 5.x / 1.x                              |
| Audio        | WaveSurfer.js                    | 7.x                                    |
| Icons        | Lucide Svelte                    | latest                                 |
| Validation   | Zod                              | 3.x                                    |

## Project Structure

```text
src/
  main/              # Electron main process (Node.js)
    index.ts          # Entry point, window creation, menu, protocol registration
    birda/            # CLI integration (spawns birda process, parses NDJSON)
    db/               # SQLite database layer (schema, migrations, CRUD modules)
    ipc/              # IPC handler modules (one per domain)
    labels/           # Species name localization service
  preload/
    index.ts          # contextBridge: exposes window.birda with allowlisted channels
  renderer/           # Svelte 5 frontend (browser context)
    index.html        # HTML entry with Content Security Policy
    src/
      main.ts         # Renderer entry point
      app.css         # Global CSS (Tailwind + daisyUI imports)
      App.svelte      # Root component
      pages/          # Page-level components (Analysis, Detections, Map, Species, Settings)
      lib/
        components/   # Reusable UI components (PascalCase .svelte files)
        stores/       # State management (.svelte.ts files using $state runes)
        utils/        # Helpers (ipc.ts wrappers, format.ts, shortcuts.ts)
shared/
  types.ts            # TypeScript interfaces shared between main and renderer
messages/
  en.json             # i18n message catalog (Paraglide)
build/                # Electron-builder resources (icons, NSIS installer script)
```

## Path Aliases

| Alias          | Resolves To                    | Available In    |
| -------------- | ------------------------------ | --------------- |
| `$lib/*`       | `src/renderer/src/lib/*`       | Renderer only   |
| `$shared/*`    | `shared/*`                     | Main + Renderer |
| `$paraglide/*` | `src/renderer/src/paraglide/*` | Renderer only   |

## TypeScript Configuration

Two separate tsconfig files — **never mix them**:

- **`tsconfig.json`** — Renderer + Shared. Extends `@tsconfig/svelte`. Includes DOM libs, `$lib` and `$paraglide` aliases.
- **`tsconfig.node.json`** — Main + Preload + Shared. Node.js only, no DOM. Has `types: ["node"]`.

Both use: `strict: true`, `exactOptionalPropertyTypes: true`, `noEmit: true`, `moduleResolution: "bundler"`.

## Commands

Task runner: **Taskfile.yml** (Go Task) or npm scripts.

```bash
# Development
task dev                    # electron-vite dev with HMR

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tphakala/birda-gui](https://github.com/tphakala/birda-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

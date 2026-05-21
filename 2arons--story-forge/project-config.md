---
trigger: always_on
description: Story-forge is a cross-platform AI chatting application built with:
---

## Project Overview

Story-forge is a cross-platform AI chatting application built with:
- **Frontend**: Svelte 5 + TypeScript
- **Desktop**: Tauri 2.5 (Rust backend)
- **Build Tool**: Vite 8
- **Styling**: Tailwind CSS 4
- **Package Manager**: pnpm

The application allows users to chat with various AI models (OpenAI, Claude, Gemini, and more) through a single unified interface. It features a rich user interface with support for themes, plugins, custom assets, and advanced memory systems.

## Directory Structure

```
story-forge-newest/
├── src/                    # Main application source code
│   ├── ts/                 # TypeScript business logic
│   ├── lib/                # Svelte UI components
│   ├── lang/               # Internationalization (i18n)
│   ├── etc/                # Documentation and extras
│   └── test/               # Test files
├── src-tauri/              # Tauri desktop backend (Rust)
├── server/                 # Self-hosting server implementations
│   ├── node/               # Node.js server (current)
│   └── hono/               # Hono framework server (future)
├── public/                 # Static assets
├── dist/                   # Build output
├── resources/              # Application resources
└── .github/workflows/      # CI/CD pipelines
```

### Source Code Structure (`/src`)

#### `/src/ts` - TypeScript Business Logic

| Directory/File | Purpose |
|----------------|---------|
| `storage/` | Data persistence layer (database, save files, platform adapters) |
| `process/` | Core processing logic (chat, requests, memory, models) |
| `plugins/` | Plugin system (API v3.0, sandboxing, security) |
| `gui/` | GUI utilities (colorscheme, highlight, animation) |
| `drive/` | Cloud sync and backup |
| `translator/` | Translation system |
| `model/` | Model definitions and integrations |
| `sync/` | Multi-user synchronization |
| `cbs.ts` | Callback system |
| `characterCards.ts` | Character card import/export |
| `parser.svelte.ts` | Message parsing |
| `stores.svelte.ts` | Svelte stores for state management |
| `globalApi.svelte.ts` | Global API methods |
| `bootstrap.ts` | Application initialization |

#### `/src/ts/process` - Core Processing

| Directory/File | Purpose |
|----------------|---------|
| `index.svelte.ts` | Main chat processing orchestration |
| `request/` | API request handlers (OpenAI, Anthropic, Google) |
| `memory/` | Memory systems (HypaMemoryV2/V3, SupaMemory, HanuraiMemory) |
| `models/` | AI model integrations (NAI, OpenRouter, Ooba, local models) |
| `templates/` | Prompt templates and formatting |
| `mcp/` | Model Context Protocol support |
| `files/` | File handling (inlays, multisend) |
| `embedding/` | Vector embeddings |
| `lorebook.svelte.ts` | Lorebook/world info management |
| `scriptings.ts` | Scripting system |
| `triggers.ts` | Event triggers |
| `stableDiff.ts` | Stable Diffusion integration |
| `tts.ts` | Text-to-speech |

#### `/src/lib` - Svelte UI Components

| Directory | Purpose |
|-----------|---------|
| `ChatScreens/` | Chat interface components |
| `UI/` | General UI components (GUI, NewGUI, Realm) |
| `Setting/` | Settings panels |
| `SideBars/` | Sidebar components (Scripts, LoreBook) |
| `Others/` | Miscellaneous components |
| `Mobile/` | Mobile-specific UI |
| `Playground/` | Testing/playground features |
| `VisualNovel/` | Visual novel mode |
| `LiteUI/` | Lightweight UI variant |

## Building and Running

### Prerequisites

- Node.js 20.19+ or 22.12+ and pnpm
- Rust and Cargo (for Tauri builds)

### Development

```bash
# Web development server
pnpm dev

# Tauri desktop development
pnpm tauri dev
```

### Production Builds

```bash
# Web build
pnpm build

# Web build for hosting
pnpm buildsite

# Tauri desktop build
pnpm tauribuild
pnpm tauri build

# Hono server build
pnpm hono:build
```

### Type Checking

```bash
pnpm check
```

## Development Conventions

### Coding Style

- The project uses Prettier for code formatting
- Ensure code is formatted before committing

### State Management

The project uses Svelte 5 Runes system:
- `$state`, `$derived`, `$effect` for reactive state
- Svelte stores (writable, readable) in `stores.svelte.ts`

Key stores:
- `DBState` - Database state
- `selectedCharID` - Current character
- `settingsOpen`, `sideBarStore`, `MobileGUI` - UI state
- `loadedStore`, `alertStore` - Application state
- `DynamicGUI` - Responsive layout switching

### Styling & Theming

To ensure dynamic theme support across the app, always use the project's custom theme colors defined in `src/styles.css` when styling components with Tailwind CSS. If you need to check how these colors are dynamically managed or view available presets (like dark, light, cherry, etc.), reference `src/ts/gui/colorscheme.ts`. Only inspect this file when specifically working on theme-related logic.

Available custom theme colors include:
- `textcolor`, `textcolor2`
- `bgcolor`, `darkbg`, `darkbutton`, `selected`
- `borderc`, `darkborderc`
- `draculared`

You can safely apply Tailwind's opacity modifiers directly to these custom theme colors (e.g., `text-textcolor/90`, `bg-textcolor/5`, `border-textcolor/10`).

### File Naming Conventions

- `.svelte.ts` - Svelte 5 files with runes
- `.svelte` - Svelte component files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2aronS/story-forge](https://github.com/2aronS/story-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->

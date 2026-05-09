---
trigger: always_on
description: **Logically AI Research Assistant** is an Obsidian community plugin that integrates AI-powered research capabilities directly into the editor.
---

# Logically AI Research Assistant (Obsidian Plugin)

## Project Overview

**Logically AI Research Assistant** is an Obsidian community plugin that integrates AI-powered research capabilities directly into the editor.

- **Plugin ID**: `logically` (immutable after release)
- **Entry point**: `src/main.ts` (LogicallyPluginImpl extends Plugin) → compiled to `main.js`
- **UI**: Svelte components mounted in right sidebar view
- **API integration**: Calls Logically backend for completions, citations, privilege checks
- **Required release artifacts**: `main.js`, `manifest.json`, `styles.css`
- **License**: Non-commercial, no-derivatives (Afforai Inc., see LICENSE.md)

## Environment & Tooling

- **Node.js**: 24+ (LTS)
- **Package manager**: pnpm (required; `pnpm-lock.yaml` is canonical)
- **Bundler**: esbuild (via `esbuild.config.mjs`, outputs `main.js` + `styles.css`)
- **UI framework**: Svelte 4 (`src/ui/*.svelte` components)
- **Language**: TypeScript (strict mode, `tsconfig.json`)
- **Linting**: ESLint with Obsidian plugin rules
- **CI/CD**: GitHub Actions (lint on push, release on tag)

### Quick Start

```bash
pnpm install          # Install dependencies
pnpm dev              # Watch mode (rebuild on file change)
pnpm build            # Production build (minified, typecheck)
pnpm lint             # Check linting
pnpm lint:fix         # Auto-fix linting issues
pnpm dev:install      # Copy built plugin to Obsidian vault (requires OBSIDIAN_VAULT env var)
```

### DevInstall Script

The `dev:install` script copies built artifacts to your vault's plugin folder. **You must provide the vault path:**

```bash
# Via environment variable
export OBSIDIAN_VAULT="/path/to/MyVault"
pnpm dev:install

# Or as argument
pnpm dev:install -- "/path/to/MyVault"
```

**Do not hardcode vault paths in the script.** Paths are developer-specific.

## Folder Structure & Conventions

The project follows Obsidian community plugin conventions with Svelte for UI:

```
obsidian-logically-plugin/
├── src/
│   ├── main.ts                    # Plugin lifecycle (minimal)
│   ├── settings.ts                # Obsidian settings tab UI + login
│   ├── types.ts                   # All TypeScript types, enums, unions
│   ├── views/
│   │   └── researchAssistantView.ts  # Right sidebar view container
│   ├── ui/                        # Svelte components (ra- CSS prefix)
│   │   ├── ResearchAssistantRoot.svelte   # Top-level chat UI
│   │   ├── ChatInput.svelte               # Input + mode/model selectors
│   │   ├── MessageList.svelte             # Chat bubbles + citations
│   │   ├── FilePicker.svelte              # Vault file selector
│   │   ├── SourcesTable.svelte            # Citation sources display
│   │   ├── LoginPrompt.svelte             # Auth UI
│   │   ├── SettingsPanel.svelte           # In-chat settings modal
│   │   ├── UpgradeModal.svelte            # Premium upgrade prompt
│   │   └── (+ 2 unused: ModelSelector, ModeSelector)
│   ├── services/
│   │   └── logicallyApi.ts        # All API calls (single source of truth)
│   └── utils/
│       ├── env.ts                 # IS_DEV_BUILD flag
│       └── authErrors.ts          # Error formatting helpers
│
├── .github/workflows/
│   ├── lint.yml                   # Lint on push
│   └── release.yml                # Release on tag (draft release)
│
├── manifest.json                  # Plugin metadata (id, version, minAppVersion)
├── main.js                        # Bundled plugin (generated)
├── styles.css                     # Bundled styles (generated)
├── esbuild.config.mjs             # Build config
├── tsconfig.json                  # TypeScript config (strict, Node types)
├── eslint.config.mts              # ESLint rules
├── package.json                   # Scripts and dependencies
├── pnpm-lock.yaml                 # Lockfile (canonical)
├── version-bump.mjs               # Version bumper script
│
└── docs/
    ├── README.md                  # User guide (install, setup, usage)
    ├── DEVELOPMENT.md             # Developer setup & release process
    └── AGENTS.md                  # This file (contributor guidelines)
```

### Key Principles

1. **Keep `main.ts` minimal**: Only plugin lifecycle (onload, onunload, view registration, ribbon icon, commands). Delegate logic to services/UI.
2. **Single source of truth for API calls**: All HTTP requests go through `services/logicallyApi.ts`.
3. **Settings via Obsidian**: Use `this.loadData()` / `this.saveData()` (no manual JSON file handling).
4. **Component organization**: Each Svelte component has a single, clear responsibility.
5. **CSS prefix**: All UI CSS classes start with `ra-` (research assistant) to avoid Obsidian collisions.
6. **Types first**: Define all interfaces/unions in `src/types.ts` (central registry).

### File Size Guidelines

- Keep files under ~300 lines; break into smaller modules if larger.
- `main.ts` should be <150 lines.
- Services (like `logicallyApi.ts`) can be larger (domain-specific logic).

## Manifest Rules

**File**: `manifest.json`

```json
{
  "id": "logically",
  "name": "Logically AI Research Assistant",
  "version": "1.0.0",
  "minAppVersion": "1.0.0",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [afforai/obsidian-logically-plugin](https://github.com/afforai/obsidian-logically-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

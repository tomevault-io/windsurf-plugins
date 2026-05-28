---
trigger: always_on
description: > AI-powered i18n JSON translation tool with CLI and framework plugins
---

# AGENTS.md - dev-translate

> AI-powered i18n JSON translation tool with CLI and framework plugins

## Project Overview

**Name:** @aexol/dev-translate (monorepo)  
**Purpose:** Automatically translate i18n JSON files using AI via GraphQL API  
**Tech Stack:** TypeScript 5.5+, ES Modules, pnpm workspaces  
**API Endpoint:** `https://backend.devtranslate.app/graphql`

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLI / Plugins                           │
│  ┌─────────┐  ┌──────────────────┐  ┌────────────────────────┐ │
│  │   cli   │  │ vite-plugin-dev- │  │ nextjs-dev-translate-  │ │
│  │         │  │    translate     │  │       plugin           │ │
│  └────┬────┘  └────────┬─────────┘  └───────────┬────────────┘ │
│       │                │                        │               │
│       │                └────────────┬───────────┘               │
│       │                             │                           │
│       ▼                             ▼                           │
│  ┌─────────┐                   ┌─────────┐                      │
│  │ config  │                   │  watch  │                      │
│  └────┬────┘                   └────┬────┘                      │
│       │                             │                           │
│       └──────────────┬──────────────┘                           │
│                      │                                          │
│                      ▼                                          │
│                 ┌─────────┐                                     │
│                 │  core   │ ◄── GraphQL API (Zeus client)       │
│                 └─────────┘                                     │
└─────────────────────────────────────────────────────────────────┘
```

## Package Structure

### `packages/core` - @aexol/dev-translate-core

**Main translation logic and GraphQL API client**

Key exports:

- `translateLocaleFolder()` - Translate all JSON files in locale directory
- `predictLocaleFolder()` - Predict token consumption before translation
- `clearAccountCache()` - Clear translation cache for account
- `getOutputLanguages()` - Get list of output language folders
- `Languages` - Enum of supported languages
- `LogLevels` - Logging verbosity levels
- `BackendProps` - Backend configuration options type
- `LangPair` - Language/folder name pair type

Dependencies: `cross-fetch`, `p-queue`

### `packages/config` - @aexol/dev-translate-config

**Configuration file handling (.dev-translate.json)**

Key exports:

- `config(cwd)` - ConfigMaker instance for project configuration
- `ProjectOptions` - Full configuration type
- `LangPair` - Re-exported from core

Dependencies: `config-maker`, `@aexol/dev-translate-core`

### `packages/cli` - @aexol/dev-translate

**Commander-based CLI tool**

Commands:

- `translate` - Translate locale files (supports `-w` for watch mode)
- `predict` - Predict token consumption
- `clear` - Clear account cache

Dependencies: `commander`, `chalk`, `chokidar`, `@aexol/dev-translate-core`, `@aexol/dev-translate-config`, `@aexol/dev-translate-watch`

### `packages/watch` - @aexol/dev-translate-watch

**Chokidar-based file watcher for automatic translation**

Key exports:

- `watch(options)` - Start watching locale directory for changes
- `DevTranslateOptions` - Watch configuration type
- `Languages` - Re-exported from core

Dependencies: `chokidar`, `@aexol/dev-translate-core`

### `packages/vite-plugin-dev-translate` - @aexol/vite-plugin-dev-translate

**Vite plugin wrapper**

Key exports:

- `default` (devTranslatePlugin) - Vite plugin factory
- `Languages` - Re-exported from watch

Peer dependencies: `vite >=5`

### `packages/nextjs-dev-translate-plugin` - @aexol/nextjs-dev-translate-plugin

**Next.js plugin wrapper**

Key exports:

- `withDevTranslate(nextConfig, options)` - Next.js config wrapper
- `Languages` - Re-exported from watch

Peer dependencies: `next >=13`

### `packages/testground`

**Testing environment for development**

### `packages/dynamite` - @aexol/dynamite

**React i18n library with build-time string extraction and SSR/RSC support**

Key exports:

- `useDynamite()` - React hook for client-side translations
- `DynamiteProvider` - Context provider for translations
- `getDynamite()` - Async server utility for RSC
- `getDynamiteSync()` - Sync server utility
- `loadTranslations()` - Load translations from JSON files
- `extractStrings()` - Extract t() calls from source directories
- `extractStringsFromSource()` - Extract t() calls from source code

CLI Commands:

- `dynamite init` - Create .dynamite.json config file
- `dynamite extract` - Extract translation strings from source files
- `dynamite translate` - Translate extracted strings via API

Dependencies: `@aexol/dev-translate-core`, `commander`, `glob`, `typescript`

Peer dependencies: `react >=18`

## Key Types

### ProjectOptions (config)

```typescript
type ProjectOptions = BackendProps & {
  inputLanguageFolderName: string; // Folder name for source language (e.g., "en")
  inputLanguage: string; // Source language code (e.g., "ENGB")
  apiKey: string; // API key for translation service
  localeDir: string; // Directory containing locale folders
};
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aexol-studio/dev-translate](https://github.com/aexol-studio/dev-translate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->

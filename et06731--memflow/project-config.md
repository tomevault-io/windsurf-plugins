---
trigger: always_on
description: **Memflow** is a browser extension (Chrome MV3) that exports AI conversations from platforms like DeepSeek, ChatGPT, and Kimi to Obsidian in Markdown format.
---

# Memflow - Agent Development Guide

## Project Overview

**Memflow** is a browser extension (Chrome MV3) that exports AI conversations from platforms like DeepSeek, ChatGPT, and Kimi to Obsidian in Markdown format.

- **Framework**: [Plasmo](https://www.plasmo.com/) - Modern browser extension framework
- **Language**: TypeScript
- **UI**: React with inline styles (no CSS files)
- **Package Manager**: pnpm

## Build Commands

```bash
# Install dependencies
pnpm install

# Development - starts dev server with hot reload
pnpm dev

# Build production version
pnpm build

# Package extension for distribution
pnpm package
```

## Test Commands

```bash
# Run all tests once
pnpm test

# Run tests in watch mode (for development)
pnpm test:watch

# Run tests with coverage report
pnpm test:coverage
```

## Project Structure

```
src/
├── contents/           # Content Scripts (runs in web pages)
│   ├── adapters/       # Platform-specific adapters
│   │   ├── base-adapter.ts      # Abstract base class
│   │   ├── deepseek.ts          # DeepSeek adapter
│   │   ├── chatgpt.ts           # ChatGPT adapter
│   │   ├── kimi.ts              # Kimi adapter
│   │   └── index.ts             # Adapter factory
│   └── index.tsx       # Main content script (toolbar button)
├── processing/         # Data processing layer
│   ├── markdown-builder.ts      # Markdown generation
│   ├── metadata-generator.ts    # Metadata extraction
│   └── local-algorithms.ts      # Text processing algorithms
├── obsidian/           # Obsidian integration
│   └── uri-handler.ts           # Obsidian URI protocol handler
├── types/              # TypeScript type definitions
├── config/             # Configuration files
│   ├── selectors.json           # DOM selectors per platform
│   └── prompts.json             # LLM prompt templates
├── utils/              # Utility functions
│   └── cleaner.ts               # HTML/text cleaning
└── popup.tsx           # Extension popup UI
```

## Code Style Guidelines

### TypeScript

- **Strict mode**: Enabled (inherited from Plasmo tsconfig)
- **No semicolons**: Use ASI (Automatic Semicolon Insertion)
- **Double quotes**: For strings
- **No trailing commas**
- **2-space indentation**

### Imports

Sorted by `@ianvs/prettier-plugin-sort-imports`:

1. Node.js built-in modules
2. Third-party modules
3. `@plasmo/*` packages
4. `@plasmohq/*` packages
5. `~/*` project imports (path alias for root)
6. Relative imports `./` and `../`

Example:
```typescript
import type { PlasmoCSConfig } from "plasmo"
import { useEffect, useState } from "react"

import { createMarkdownBuilder } from "~/processing"
import { detectPlatformAdapter } from "./adapters"
```

### Naming Conventions

- **Files**: kebab-case (e.g., `base-adapter.ts`, `markdown-builder.ts`)
- **Classes**: PascalCase (e.g., `BaseAdapter`, `MarkdownBuilder`)
- **Interfaces**: PascalCase with `I` prefix (e.g., `IAdapter`)
- **Functions**: camelCase, descriptive names
- **Constants**: UPPER_SNAKE_CASE for true constants
- **Types**: PascalCase

### Error Handling

- Always use try/catch in async functions
- Provide user-friendly error messages in Chinese
- Log errors with emoji prefixes for visibility:
  - `console.log("✅ Success message")`
  - `console.warn("⚠️ Warning message")`
  - `console.error("❌ Error message")`

Example:
```typescript
try {
  const result = await someAsyncOperation()
  console.log("✅ 操作成功")
} catch (error) {
  console.error("❌ 操作失败:", error)
  showToast(`导出失败: ${error.message}`, "error")
}
```

### Platform Adapter Pattern

When adding a new AI platform adapter:

1. Create a new file in `src/contents/adapters/{platform-name}.ts`
2. Extend `BaseAdapter` and implement required methods
3. Add selectors to `src/config/selectors.json`
4. Register in `src/contents/adapters/index.ts`

Template:
```typescript
import { BaseAdapter } from "./base-adapter"
import type { SelectorConfig } from "./base-adapter"
import selectors from "../../config/selectors.json"

export class NewPlatformAdapter extends BaseAdapter {
  platformName = "NewPlatform"
  selectors: SelectorConfig = selectors.platforms.newplatform as SelectorConfig

  detectPlatform(): boolean {
    return window.location.host.includes("platform.com")
  }
}

export function createNewPlatformAdapter(): NewPlatformAdapter {
  return new NewPlatformAdapter()
}
```

### DOM Selectors

All DOM selectors are centralized in `src/config/selectors.json`. Use multiple selectors as fallbacks:

```json
{
  "platformName": {
    "messageContainer": "selector1, selector2, selector3",
    "userMessage": "[data-role='user'], .user-message",
    "aiMessage": "[data-role='assistant'], .assistant-message"
  }
}
```

### Content Script Configuration

Content scripts target specific URLs via `config` export:

```typescript
export const config: PlasmoCSConfig = {
  matches: [
    "https://chat.deepseek.com/*",
    "https://*.deepseek.com/*"
  ]
}
```

### UI Components

- Use inline styles (no CSS files)
- Dark theme with amber (#f59e0b) accents
- Font: JetBrains Mono for monospace, Cinzel for headings
- Toast notifications for user feedback

### State Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ET06731/Memflow](https://github.com/ET06731/Memflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

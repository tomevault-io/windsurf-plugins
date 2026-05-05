---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📚 Essential Documentation

**🎯 BEFORE starting ANY development task, please review these documents:**

- **[UI Design Guide](./docs/UI-DESIGN-GUIDE.md)** - Complete visual design system, color schemes, component specifications, and responsive design guidelines
- **[Development Guide](./docs/DEVELOPMENT-GUIDE.md)** - Code style standards, file organization, Chrome extension patterns, and best practices
- **[Icon Design Prompts](./docs/ICON-DESIGN-PROMPTS.md)** - Professional icon design prompts for brand identity, optimized for Apple Design aesthetics and multiple sizes

These guides contain critical lessons learned and established patterns that ensure consistency and prevent common pitfalls.

## Project Overview

Feishu Converter is a Chrome browser extension (also available for Edge and Firefox) that converts Lark/Feishu documents to Markdown format. The project supports downloading, copying, and previewing documents with WeChat formatting capabilities.

## Development Commands

### Essential Commands
```bash
# Install dependencies (run from root)
pnpm i

# Build the entire project
pnpm run build

# Build only the Chrome extension
cd apps/chrome-extension && pnpm run build

# Development build (non-minified)
cd apps/chrome-extension && pnpm run build:dev

# Type checking
pnpm run type-check

# Linting
pnpm run lint

# Format code
pnpm run format

# Run tests
pnpm run test

# Clean build artifacts
pnpm run clean
```

### Chrome Extension Specific
```bash
# Build Firefox version
cd apps/chrome-extension && pnpm run build:firefox

# Copy assets to dist folder
cd apps/chrome-extension && npm run copy
```

## Architecture Overview

### Monorepo Structure
This is a pnpm workspace monorepo with Turbo for build orchestration:
- **apps/chrome-extension**: Main Chrome extension application
- **packages/common**: Shared utilities (DOM manipulation, time, image processing)
- **packages/lark**: Core Lark document processing and conversion logic
- **packages/rollup-config**: Shared Rollup build configuration
- **packages/typescript-config**: Shared TypeScript configurations

### Chrome Extension Architecture

The extension follows Chrome Manifest V3 architecture with distinct runtime contexts:

#### Core Components
1. **Background Script** (`src/background.ts`): Service worker handling context menus, tab management, and message routing
2. **Content Script** (`src/content.ts`): Injected into Lark pages for message passing between page and extension
3. **Sidebar Panel**: Main UI using Chrome's sidePanel API with multiple views:
   - Main sidebar (`src/sidebar/sidebar.ts`)
   - Settings page (`src/sidebar/settings.ts`)
   - WeChat preview (`src/sidebar/wechat-preview.ts`)
4. **Injected Scripts** (`src/scripts/`): Run in MAIN world context to access Lark's document APIs

#### Message Passing System
- Uses Chrome runtime messaging between background, content script, and sidebar
- window.postMessage for communication between injected scripts and content script
- Message types defined in `src/common/message.ts` with typed interfaces

#### Supported Document Sources
- *.feishu.cn, *.feishu.net, *.larksuite.com
- *.feishu-pre.net, *.larkoffice.com, *.larkenterprise.com

### Document Processing Pipeline

1. **Extraction**: Injected scripts access Lark's internal document structure
2. **Conversion**: `@dolphin/lark` package converts to MDAST (Markdown AST)
3. **Serialization**: MDAST converted to Markdown using mdast-util-to-markdown
4. **Output**: Download as file, copy to clipboard, or preview with WeChat formatting

### WeChat Integration
- MD2WeChat API integration for WeChat article formatting
- Custom CSS styling matching WeChat's design system
- Full-screen preview with floating toolbar interface
- Copy functionality optimized for WeChat backend editor compatibility

## Key Technical Details

### Build System
- **Rollup** for bundling with format-specific outputs:
  - ESM format for background/content scripts
  - IIFE format for UI scripts and injected scripts
- **Babel** for browser compatibility
- **TypeScript** with multiple tsconfig files for different runtime contexts

### Styling & Design
- Primary color scheme: `#d97759` (warm orange)
- CSS custom properties for consistent theming
- Full-screen responsive design for WeChat preview
- Modern UI with backdrop blur effects and smooth animations

### Internationalization
- i18next for translations
- Supports English (`en`) and Chinese (`zh_CN`)
- Chrome extension `_locales` structure

### State Management
- Chrome storage API for user preferences (API keys, themes, font sizes)
- No external state management library; uses local component state

## Development Notes

### File Naming Conventions
- TypeScript files use `.ts` extension throughout
- Build outputs generate corresponding `.js` files in `bundles/` directory
- HTML files are copied directly to `dist/` during build

### Extension Permissions
- contextMenus, scripting, sidePanel, tabs, storage
- Host permissions for all Lark/Feishu domains
- Content scripts inject only on document_end

### Testing
- Vitest for unit tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geekjourneyx/feishu-md2wechat](https://github.com/geekjourneyx/feishu-md2wechat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

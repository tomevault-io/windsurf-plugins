---
trigger: always_on
description: handleStorageChanges({
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the documentation website for "空荧酒馆" (Kongying Tavern), an open-source Genshin Impact map application. The site is built with VitePress and supports multiple languages (Chinese, English, Japanese).

## Development Commands

Essential commands for development:

```bash
# Setup (requires Node.js v18.0.0+)
corepack enable
pnpm i

# Development
pnpm run dev              # Start dev server
pnpm run build            # Build for production
pnpm run serve            # Preview production build

# Data and Content
pnpm run build-data       # Refresh all data (blog + member lists)
pnpm run build-member     # Refresh member list data only
pnpm run build-emoji      # Build emoji data

# Code Quality
pnpm run lint             # Run ESLint + Chinese text linting
pnpm run lint:eslint-fix  # Auto-fix ESLint issues
pnpm run lint:zh-fix      # Auto-fix Chinese text issues
pnpm run typecheck        # TypeScript type checking

# Testing and Quality
pnpm run changelog        # Generate conventional changelog
pnpm run commit           # Interactive conventional commits

# Specialized builds
pnpm run build-mpa        # Build as Multi-Page Application
```

## Import Path Configuration

The project uses the following TypeScript path mapping configuration:

```json
{
  "@/*": ["./.vitepress/theme/*"],
  "#theme/*": ["./.vitepress/theme/*"],
  "~/*": ["./src/*"],
  "vite": ["./node_modules/vite"]
}
```

**Important Import Rules:**

- Use `@/` for theme-related imports (components, stores, utils in `.vitepress/theme/`)
- Use `~/` for source content imports (components, services, types in `src/`)
- Common mistakes to avoid:
  - ❌ `import { useUserAuthStore } from '@/.vitepress/theme/stores/useUserAuth'`
  - ✅ `import { useUserAuthStore } from '@/stores/useUserAuth'`
  - ❌ `import { BlogDraft } from 'src/services/blogDraftDB'`
  - ✅ `import { BlogDraft } from '~/services/blogDraftDB'`

## Architecture

### Core Structure

- **`.vitepress/`** - VitePress configuration and theme customization
  - `config.ts` - Main VitePress configuration
  - `theme/` - Custom theme components and styles
  - `locales/` - Internationalization configurations
- **`src/`** - Content source files
  - `zh/`, `en/`, `ja/` - Localized content directories
  - `components/` - Vue components (forum, team, release pages)
  - `public/` - Static assets (images, fonts, emojis)
  - `_data/` - JSON data files

### Key Features

- **Multi-language support** with locale-specific routing
- **Forum system** with Vue components for community interaction
- **Custom emoji system** with categorized emoji collections
- **Team and staff pages** with member data management
- **VitePress-based documentation** with custom markdown extensions

### Development Notes

- Uses **pnpm** as package manager with workspace configuration
- **UnoCSS** for utility-first styling
- **TypeScript** throughout the codebase
- **Vue 3** for interactive components
- Custom markdown plugins for enhanced content formatting
- Font optimization with Fontaine
- Chinese text linting with `zhlint`

### Content Management

- Blog posts and member data are refreshed via build scripts
- Emoji data is processed from `/src/public/emojis/` directory structure
- Static assets organized by language in `/src/public/imgs/`
- Translation management with Lunaria integration

### Modern Forum Architecture (2024 Refactoring)

The forum system underwent a major architectural refactoring from factory pattern to composition-based architecture:

#### New Store Architecture

- **Page-specific stores**: Each page has isolated state (`useForumHomeStore`, `useForumUserStore`)
- **Core functional stores**: Modular stores by domain (`useForumSearchState`, `useForumViewState`, `useForumRouteState`)
- **Composition over inheritance**: Replaced complex factory patterns with composables
- **Performance optimization**: Intelligent caching, batch updates, debounced operations

#### Service Layer

- **`ForumBusinessLogic`** - Unified business logic service
- **`SimpleEventManager`** - Event-driven state synchronization
- **`SimpleCrossPageSync`** - Cross-page and cross-tab state sync via localStorage

#### Key Patterns

- Event-driven updates with deduplication (1000ms global, 500ms store-level)
- Three synchronized data arrays per store: `data`, `userSubmittedTopic`, `pinnedTopicsData`
- Hidden vs closed topic state management with different removal behaviors
- Performance monitoring with `useForumPerformanceMonitor`

## Authentication & Authorization Architecture

The project implements a comprehensive authentication system with the following components:

### Core Authentication Files

- **`.vitepress/theme/stores/useUserAuth.ts`** - Main Pinia store for authentication state management
- **`.vitepress/theme/hooks/useLogin.ts`** - Login flow management and OAuth integration
- **`.vitepress/theme/utils/auth-helpers.ts`** - Unified authentication utility functions
- **`.vitepress/theme/utils/auth-logger.ts`** - Centralized logging system for auth operations
- **`.vitepress/theme/utils/auth-errors.ts`** - Standardized error handling for auth operations

### Authentication Flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kongying-tavern/docs](https://github.com/kongying-tavern/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

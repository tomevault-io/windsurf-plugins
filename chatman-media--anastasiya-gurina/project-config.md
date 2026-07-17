---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **monorepo** containing:
1. **Photography Portfolio** - Website showcasing Anastasiya Gurina's photography
2. **Telegram Mini App** - NFT marketplace for purchasing exclusive photos
3. **Backend API** - Hono-based API for NFT and payment management
4. **Smart Contracts** - TON blockchain contracts for NFT minting
5. **Shared Packages** - UI components and TypeScript configurations

Built with Bun Workspaces, Turborepo, React, Next.js, and TON blockchain.

## Development Commands

### Package Manager
This project uses **Bun** as its package manager. Always use `bun` instead of `npm`, `yarn`, or `pnpm`.

### Monorepo Commands (from root)
- `bun install` - Install all dependencies
- `bun run dev` - Start all apps in development mode
- `bun run build` - Build all packages with Turborepo
- `bun run build:portfolio` - Build only portfolio app
- `bun run build:telegram` - Build only Telegram Mini App
- `bun run build:backend` - Build only backend API
- `bun run build:contracts` - Build only smart contracts
- `bun run type-check` - Run TypeScript type checking across all packages
- `bun run test` - Run tests across all packages
- `bun run clean` - Clean all build artifacts

### App-Specific Commands
Navigate to specific app directory first:

**Portfolio** (`cd apps/portfolio`)
- `bun run dev` - Start Vite dev server (port 5173)
- `bun run build` - Build portfolio
- `bun run preview` - Preview production build

**Telegram Mini App** (`cd apps/telegram-mini-app`)
- `bun run dev` - Start Next.js dev server (port 3000)
- `bun run dev:https` - Start with HTTPS for Telegram testing
- `bun run build` - Build Next.js app
- `bun run start` - Start production server

**Backend** (`cd packages/backend`)
- `bun run dev` - Start API with hot reload (port 8000)
- `bun run build` - Build backend
- `bun run start` - Start production server

**Contracts** (`cd packages/contracts`)
- `bun run build` - Compile FunC contracts
- `bun run test` - Run contract tests
- `bun run deploy` - Deploy to TON blockchain

## Monorepo Architecture

### Workspace Structure
This monorepo uses Bun Workspaces with Turborepo for build orchestration:

```
/
├── apps/
│   ├── portfolio/          # Vite + React portfolio
│   └── telegram-mini-app/  # Next.js Telegram app
├── packages/
│   ├── ui/                 # Shared UI components
│   ├── typescript-config/  # Shared TS configs
│   ├── backend/            # Hono API server
│   └── contracts/          # TON smart contracts
├── package.json            # Root workspace
├── turbo.json             # Turborepo config
└── bun.lock
```

### Dependency Graph
- `telegram-mini-app` depends on `@workspace/ui`
- `telegram-mini-app` calls `backend` API
- `backend` uses `@workspace/contracts`
- `portfolio` uses `@workspace/ui`
- All apps extend `@workspace/typescript-config`

### Apps Overview

#### 1. Portfolio (`apps/portfolio`)
Vite + React photography portfolio with:

- **App.tsx** - Main application component that composes all sections and wraps them with context providers (ThemeProvider, LayoutProvider)
- **Header** - Navigation bar with language switcher, theme switcher, and layout switcher
- **Hero, About, PhotoGallery, Contact, Footer** - Page sections rendered in order
- **ui/** - Reusable UI components (Button, DropdownMenu) using Radix UI primitives

### Context Providers Architecture
The app uses React Context for global state management:

1. **ThemeProvider** (src/components/theme-provider.tsx)
   - Manages theme state: 'light', 'dark', or 'system'
   - Persists theme preference to localStorage (key: 'vite-ui-theme')
   - Uses a ThemeWrapper component that applies theme classes after client-side mount
   - Detects system theme using `window.matchMedia('(prefers-color-scheme: dark)')`
   - Export: `useTheme()` hook for accessing theme state

2. **LayoutProvider** (src/components/layout-switcher.tsx)
   - Manages layout width state: default or full-width
   - Persists layout preference to localStorage (key: 'layout-width')
   - Applies 'layout-full-width' class to document.body when full-width is enabled
   - Export: `useLayout()` hook and `LayoutSwitcher` component

### i18n (Internationalization)
- Configuration: src/lib/i18n.ts
- Translation files: src/locales/en/translation.json and src/locales/ru/translation.json
- Default language: English ('en')
- Access translations using `useTranslation()` hook from react-i18next
- When adding new translatable strings, update both translation files

### Photo Gallery
- Photos are stored in public/photos/ directory
- Photo paths are constructed using `import.meta.env.BASE_URL` to handle GitHub Pages base path
- The base path is set to "/Anastasiya-Gurina/" in vite.config.ts for GitHub Pages deployment
- Photo list is hardcoded in App.tsx (portfolioPhotos array)
- Uses react-photo-album library for gallery layout

### Path Aliasing
- `@/*` is aliased to `src/*` (configured in vite.config.ts and tsconfig.json)
- Always use `@/` imports for internal modules: `import { cn } from "@/lib/utils"`

### Styling
- Tailwind CSS v4 with @tailwindcss/vite plugin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chatman-media/Anastasiya-Gurina](https://github.com/chatman-media/Anastasiya-Gurina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->

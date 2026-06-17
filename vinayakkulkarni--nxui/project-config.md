---
trigger: always_on
description: > **For AI Assistants (Claude Code, Cursor, etc.)**
---

# CLAUDE.md - nxui Development Guide

> **For AI Assistants (Claude Code, Cursor, etc.)**
> This is the single source of truth for the nxui project. When this file conflicts with `.agents/skills/`, **this file wins**.

---

## Quick Reference

```bash
pnpm install            # Install dependencies
pnpm run dev            # Start dev server (localhost:3000)
pnpm run build          # Production build (Cloudflare Pages)
pnpm run lint           # Run oxlint
pnpm run lint:fix       # Fix lint issues
pnpm run format         # Format with oxfmt
pnpm run format:check   # Check formatting
pnpm run build:registry # Build registry JSONs → public/r/
pnpm run update:deps    # Interactively update all deps (taze)
```

---

## Project Overview

**nxui** is an exact Vue port of [componentry.fun](https://www.componentry.fun) — a beautiful animated UI component library. Users copy, paste, and ship animated components via the `shadcn-vue` CLI.

- **Domain**: `nxui.geoql.in` (NOT nxui.dev)
- **Port target**: React/Framer Motion → Vue 3/motion-v
- **Deploy**: Cloudflare Pages with D1 database for content

---

## Tech Stack

| Layer           | Technology                                       | Version                                                           |
| --------------- | ------------------------------------------------ | ----------------------------------------------------------------- |
| Framework       | Nuxt 4                                           | v4.4.2 (`future.compatibilityVersion: 4`)                         |
| UI              | Vue 3 Composition API                            | v3.5+                                                             |
| Styling         | Tailwind CSS v4                                  | `@nuxtjs/tailwindcss` v7 (wraps `@tailwindcss/vite` — no PostCSS) |
| Animation       | motion-v                                         | v2.0.1 (Vue Framer Motion equivalent)                             |
| Utilities       | VueUse                                           | v14.2.1                                                           |
| Content         | @nuxt/content                                    | v3                                                                |
| Icons           | @nuxt/icon (Iconify)                             | v2.2.1 (lucide:\_, simple-icons:\_)                               |
| Color Mode      | @nuxtjs/color-mode                               | v4                                                                |
| Headless UI     | reka-ui                                          | v2.9.2                                                            |
| Highlighting    | shiki                                            | v4.0.2                                                            |
| Class Utils     | class-variance-authority + clsx + tailwind-merge |                                                                   |
| Types           | TypeScript                                       | v6.0+ strict                                                      |
| Package Manager | pnpm                                             | v11.3.0                                                           |

---

## Project Structure

```
nxui/
├── CLAUDE.md                          # THIS FILE
├── nuxt.config.ts                     # Nuxt config
├── package.json
├── app/
│   ├── assets/css/main.css            # Tailwind v4 + oklch CSS vars
│   ├── components/
│   │   ├── content/                   # Demo wrappers (ComponentDemo, Demo*.vue)
│   │   ├── docs/                      # Docs layout components
│   │   └── ui/                        # Shadcn-vue base components
│   ├── composables/                   # Vue composables (kebab-case files)
│   ├── layouts/                       # Nuxt layouts (docs.vue)
│   ├── lib/utils.ts                   # cn() utility
│   ├── pages/                         # File-based routing
│   └── types/                         # ALL types live here
│       └── components.ts              # Component interfaces
├── registry/
│   └── new-york/                      # Registry components (NOT auto-imported)
│       ├── shimmer-button/
│       │   ├── ShimmerButton.vue
│       │   └── index.ts
│       ├── node-diagram/
│       │   ├── NodeDiagram.vue
│       │   ├── NodeDiagramNode.vue
│       │   ├── NodeDiagramTrace.vue
│       │   └── index.ts
│       └── ...
├── content/                           # @nuxt/content markdown
│   ├── docs/components/               # Component doc pages
│   └── 3.components/                  # Numbered component pages
├── public/r/                          # Built registry JSON files
├── scripts/build-registry.ts          # Registry build script
└── .agents/skills/                    # AI skills (supplementary)
```

---

## Critical Rules

### Rule 1: Types NEVER Inline

NEVER define `interface` or `type` inside `.vue` files or composables. ALL types go in `app/types/`.

```typescript
// ❌ WRONG — inside a .vue file
interface MyProps {
  name: string;
}

// ✅ CORRECT — in app/types/components.ts
export interface MyProps {
  name: string;
}
// Import: import type { MyProps } from '~/types/components';
```

### Rule 2: No `any` — Ever


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinayakkulkarni/nxui](https://github.com/vinayakkulkarni/nxui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

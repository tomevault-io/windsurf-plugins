---
trigger: always_on
description: Hono UI is a **shadcn-style component library and CLI** specifically built for **Hono-first stacks**. It provides SSR-first UI primitives, paid blocks, and starter kits for developers building applications with Hono (deployed to Bun, Deno, Cloudflare Workers, or Node).
---

# Hono UI - Agent Instructions

## Product Overview

Hono UI is a **shadcn-style component library and CLI** specifically built for **Hono-first stacks**. It provides SSR-first UI primitives, paid blocks, and starter kits for developers building applications with Hono (deployed to Bun, Deno, Cloudflare Workers, or Node).

**Core value proposition:**
- Free, copy/paste primitives that work with Hono's JSX renderer (no React)
- Paid UI blocks and starter kits for rapid application development
- Zero client JavaScript by default, optional progressive enhancement via `@hono-ui/enhance`
- Edge-friendly, SSR-first architecture

**Target users:**
- Indie developers building Hono apps who want polished UI quickly
- AI app builders (chat interfaces, prompt studios, usage dashboards) shipping MVPs
- Developers who prefer Hono runtimes and minimal client-side JavaScript

**Business model:**
- Free: CLI, ~25 primitives, ~20 blocks, documentation
- Pro Blocks ($99): 100+ marketing, dashboard, and AI UI blocks
- Starters Bundle ($249): Pro Blocks + AI SaaS Starter + Dashboard Admin Starter
- Optional $79/year for continued updates after year 1
- Payments via Lemon Squeezy, license keys validated by registry API

---

## Core Commands

```bash
# Install dependencies (monorepo)
pnpm install

# Build all packages
pnpm build

# Run tests
pnpm test

# Type check
pnpm typecheck

# Lint
pnpm lint

# Development mode (docs site)
pnpm dev

# Build CLI locally
pnpm --filter @hono-ui/cli build

# Build enhance package
pnpm --filter @hono-ui/enhance build

# Deploy registry API to Cloudflare
pnpm --filter @hono-ui/registry deploy
```

---

## Project Layout

```
hono-ui/
├── packages/
│   ├── cli/                      # `hono-ui` CLI tool
│   │   ├── src/
│   │   │   ├── commands/
│   │   │   │   ├── init.ts       # `npx @hono-ui/cli init` - project setup
│   │   │   │   ├── add.ts        # `npx @hono-ui/cli add` - add components/blocks
│   │   │   │   └── diff.ts       # `npx @hono-ui/cli diff` - check for updates
│   │   │   ├── utils/
│   │   │   │   ├── registry.ts   # Fetch from registry API
│   │   │   │   ├── config.ts     # Read/write hono-ui.json
│   │   │   │   └── files.ts      # File system operations
│   │   │   └── index.ts          # CLI entry point
│   │   └── package.json
│   │
│   ├── enhance/                  # `@hono-ui/enhance` - optional JS
│   │   ├── src/
│   │   │   ├── dialog.ts         # Dialog open/close
│   │   │   ├── dropdown.ts       # Dropdown toggle
│   │   │   ├── toast.ts          # Toast notifications
│   │   │   ├── clipboard.ts      # Copy to clipboard
│   │   │   ├── theme.ts          # Theme toggle (dark/light)
│   │   │   ├── sidebar-mobile.ts # Mobile sidebar slide-in
│   │   │   └── index.ts          # Package exports
│   │   └── package.json
│   │
│   └── registry/                 # Registry API (Cloudflare Worker)
│       ├── src/
│       │   ├── index.ts          # Hono app entry point
│       │   ├── routes/
│       │   │   ├── components.ts # GET /r/:name - serve components
│       │   │   └── auth.ts       # License key validation
│       │   └── middleware/
│       │       └── auth.ts       # Auth middleware for paid content
│       ├── wrangler.toml         # Cloudflare Worker config
│       └── package.json
│
├── registry/                     # Component source files (served by API)
│   ├── ui/                       # Free primitives
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── textarea.tsx
│   │   ├── card.tsx
│   │   ├── badge.tsx
│   │   ├── alert.tsx
│   │   ├── label.tsx
│   │   ├── separator.tsx
│   │   ├── skeleton.tsx
│   │   ├── spinner.tsx
│   │   ├── table.tsx
│   │   ├── checkbox.tsx
│   │   ├── radio-group.tsx
│   │   ├── switch.tsx
│   │   ├── select.tsx
│   │   ├── container.tsx
│   │   ├── stack.tsx
│   │   ├── grid.tsx
│   │   ├── stat.tsx
│   │   ├── progress.tsx
│   │   ├── empty-state.tsx
│   │   ├── breadcrumbs.tsx
│   │   ├── pagination.tsx
│   │   ├── tabs.tsx
│   │   ├── form-field.tsx
│   │   ├── sidebar-menu-button.tsx
│   │   └── sidebar-item.tsx
│   │
│   ├── blocks/                   # UI blocks (free + paid)
│   │   ├── free/
│   │   │   ├── hero-simple.tsx
│   │   │   ├── hero-centered.tsx
│   │   │   ├── feature-grid.tsx
│   │   │   ├── cta-simple.tsx
│   │   │   ├── footer-simple.tsx
│   │   │   └── ...
│   │   └── pro/                  # Paid blocks (require license)
│   │       ├── marketing/
│   │       │   ├── hero-split.tsx
│   │       │   ├── testimonials-carousel.tsx
│   │       │   ├── pricing-table.tsx
│   │       │   └── ...
│   │       ├── dashboard/
│   │       │   ├── sidebar-layout-seamless.tsx
│   │       │   ├── sidebar-layout-bordered.tsx
│   │       │   ├── sidebar-layout-inset.tsx
│   │       │   ├── sidebar-layout-floating.tsx
│   │       │   ├── sidebar-layout-simple.tsx
│   │       │   ├── header-dashboard.tsx
│   │       │   ├── page-header-simple.tsx
│   │       │   ├── page-header-actions.tsx
│   │       │   ├── page-header-stats.tsx
│   │       │   ├── page-header-tabs.tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hono-ui/hono-ui](https://github.com/hono-ui/hono-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->

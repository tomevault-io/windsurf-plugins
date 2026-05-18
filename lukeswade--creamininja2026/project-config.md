---
trigger: always_on
description: CreamiNinja is a full-stack Progressive Web App (PWA) for sharing Ninja CREAMi recipes. It enables users to create, share, and discover recipes with social features including friend connections ("Ninjagos"), privacy controls, and AI-powered recipe generation.
---

# CreamiNinja — Copilot Instructions

## Project Overview

CreamiNinja is a full-stack Progressive Web App (PWA) for sharing Ninja CREAMi recipes. It enables users to create, share, and discover recipes with social features including friend connections ("Ninjagos"), privacy controls, and AI-powered recipe generation.

**Target deployment:** Cloudflare-native stack with Pages (frontend) and Workers (API).

## Architecture

This is a **monorepo** using pnpm workspaces with two main applications:

- **`apps/api`** — Backend API (Cloudflare Workers with Hono framework)
- **`apps/web`** — Frontend PWA (React + Vite + Tailwind CSS)

### Backend (`apps/api`)
- **Framework:** Hono (web framework for Cloudflare Workers)
- **Database:** Cloudflare D1 (SQLite-compatible)
- **Storage:** Cloudflare R2 (S3-compatible for images)
- **Auth:** Email/password with server-side sessions (httpOnly cookies)
- **Security:** CSRF protection, Turnstile bot protection, rate limiting
- **AI Integration:** OpenAI API for recipe generation (abstracted provider interface)

### Frontend (`apps/web`)
- **Framework:** React 18 with TypeScript
- **Build Tool:** Vite
- **Styling:** Tailwind CSS
- **Routing:** React Router v6
- **State Management:** TanStack Query (React Query)
- **PWA:** vite-plugin-pwa with service worker support
- **Icons:** Lucide React

## Tech Stack

### Common
- **Language:** TypeScript (strict mode enabled)
- **Package Manager:** pnpm 9.0.0 (pinned via packageManager field)
- **Node Version:** 20+
- **Linting:** ESLint 9 with flat config
- **Formatting:** Prettier

### API Dependencies
- `hono` — Web framework
- `@hono/zod-validator` — Request validation
- `zod` — Schema validation
- `jose` — JWT/session token handling
- `aws4fetch` — AWS signature v4 for R2 presigned URLs
- `nanoid` — ID generation
- `@cloudflare/workers-types` — TypeScript types for Workers

### Web Dependencies
- `react` & `react-dom` — UI framework
- `@tanstack/react-query` — Data fetching and caching
- `react-router-dom` — Client-side routing
- `clsx` — Conditional class names
- `lucide-react` — Icon library
- `tailwindcss` — Utility-first CSS
- `vite-plugin-pwa` — PWA capabilities

## Coding Conventions

### General Style
- **Indentation:** 2 spaces (enforced by `.editorconfig`)
- **Quotes:** Double quotes for strings
- **Semicolons:** Always use semicolons
- **Trailing commas:** None (per Prettier config)
- **Line endings:** LF (Unix-style)
- **Final newline:** Always include

### TypeScript
- **Strict mode:** Enabled in all packages
- **Type imports:** Use `import type` for type-only imports
- **No any:** Avoid `any` type; use `unknown` or proper typing
- **Unused vars:** Prefix with underscore (`_`) if intentionally unused
- **Module resolution:** Bundler mode (for Vite/Wrangler compatibility)

### React/Frontend
- **Components:** Use functional components with hooks (no class components)
- **File organization:** Group by feature/route in `src/routes` and `src/components`
- **Styling:** Use Tailwind utility classes; use `clsx` for conditional classes
- **State:** Prefer TanStack Query for server state; React hooks for local state
- **Exports:** Default exports for route components; named exports for utilities

### API/Backend
- **Framework:** Use Hono patterns (middleware, context)
- **Validation:** Use Zod schemas with `@hono/zod-validator`
- **Error handling:** Return JSON with `{ ok: false, error: { message: "..." } }`
- **Database:** Use parameterized queries to prevent SQL injection
- **Auth:** Session tokens in httpOnly cookies with CSRF protection
- **File naming:** Kebab-case for route files, camelCase for utility modules

## Development Workflow

### Setup
```bash
# Root install
pnpm install

# API setup
cd apps/api
pnpm db:local:setup    # Creates local D1 database
pnpm db:seed:local     # Optional: Add demo data

# Web setup
cd apps/web
# Optional: Create .env to override defaults
# VITE_API_BASE defaults to http://localhost:8787
# VITE_TURNSTILE_SITE_KEY is optional for local dev (API can bypass with TURNSTILE_BYPASS=true)
```

### Running Locally
```bash
# From root - run all apps
pnpm dev

# Or individually
cd apps/api && pnpm dev   # API: http://localhost:8787
cd apps/web && pnpm dev   # Web: http://localhost:5173
```

### Linting & Formatting
```bash
# Lint all packages
pnpm lint

# Format all packages
pnpm format

# Individual packages
cd apps/api && pnpm lint
cd apps/web && pnpm format
```

### Building
```bash
# Build all packages
pnpm build

# Individual packages
cd apps/api && pnpm deploy    # Deploy API to Cloudflare
cd apps/web && pnpm build     # Build for Cloudflare Pages
```

## Database & Migrations

- **Schema location:** `apps/api/migrations/`
- **Migration tool:** Wrangler D1
- **Local database:** `.wrangler/state/v3/d1/miniflare-D1DatabaseObject/`
- **Commands:**
  - `pnpm db:local:setup` — Create and migrate local DB
  - `pnpm db:remote:apply` — Apply migrations to remote D1
  - `pnpm db:seed:local` / `pnpm db:seed:remote` — Seed demo data


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukeswade/creamininja2026](https://github.com/lukeswade/creamininja2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

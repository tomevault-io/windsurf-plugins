---
trigger: always_on
description: Personal website and blog for Piyush Mehta (`piyushmehta.com`) — a portfolio, technical blog, newsletter platform, and consulting landing page. Built with **Astro 7.x** (SSR, server output), **TypeScript 6.x**, **React 19** islands, **MDX** blog content, **Tailwind CSS v4**, deployed to **Vercel** with ISR (24h expiration).
---

# Repository Guidelines

## Project Overview

Personal website and blog for Piyush Mehta (`piyushmehta.com`) — a portfolio, technical blog, newsletter platform, and consulting landing page. Built with **Astro 7.x** (SSR, server output), **TypeScript 6.x**, **React 19** islands, **MDX** blog content, **Tailwind CSS v4**, deployed to **Vercel** with ISR (24h expiration).

Key capabilities:
- Blog with MDX content, syntax-highlighted code blocks, Giscus comments
- Social card / OG image generation pipeline (satori + resvg-js)
- Newsletter subscription (Resend + Upstash Redis + PostgreSQL)
- Contact form with CSRF protection and rate limiting
- Blog post reactions (Upstash Redis)
- Global search via Pagefind
- Accessibility-verified design
- Sentry error monitoring (client + server)
- Full RSS feed, sitemap, robots.txt

---

## Architecture & Data Flow

### Rendering Model

```
Browser ──► Astro SSR (Vercel) ──► React Islands (client:load/visible/idle)
                │
                ├── MDX Content Collections ──► src/content/blog/**/*.mdx
                ├── API Endpoints ──► src/pages/api/*.ts
                └── Middleware ──► Security headers, OG caching
```

- **Astro pages** render server-side, produce static HTML with optional hydration directives for React islands
- **React islands** (`*.tsx`) hydrate on the client via `client:load`, `client:visible`, or `client:idle` directives
- **API routes** (`src/pages/api/*.ts`) are serverless functions — all set `prerender = false`
- **Middleware** (`src/middleware/index.ts`) chains security headers and OG cache via `astro/middleware` `sequence`

### Data Dependencies

| Service | Usage | Library |
|---------|-------|---------|
| **Upstash Redis** | Rate limiting, reaction counters, newsletter rate limiting | `@upstash/redis`, `@upstash/ratelimit` |
| **ioredis** | Fallback Redis client (when Upstash unavailable) | `ioredis` |
| **PostgreSQL** | Newsletter subscriber persistence | `pg` |
| **Resend** | Transactional email (contact form, newsletter confirmation) | `resend` |
| **GitHub API** | Project repository data for showcase | `fetch` (native) |
| **Sentry** | Error monitoring (client + server) | `@sentry/astro` |
| **Vercel Speed Insights** | Performance analytics | `@vercel/speed-insights` |

### Build Pipeline

```
scripts/build.mjs orchestrates:
  1. bunx varlock typegen         — Generate env type definitions
  2. migrate-images-to-public     — Copy blog images → public/
  3. astro build                  — Main Astro SSR build
  4. run-pagefind                 — Search index over dist/client/
  5. generate-enhanced-sitemap    — Post-build sitemap.xml
  6. generate-static-rss          — Post-build rss.xml
  7. generate-resume-pdf          — Resume PDF via Playwright
```

---

## Key Directories

| Path | Purpose | Contents |
|------|---------|---------|
| `src/components/` | UI components | 28 Astro (`.astro`) + 14 React (`.tsx`) islands |
| `src/pages/` | File-based routes + API | 19 pages + 10 API endpoints + 6 special routes (RSS, sitemap, OG) |
| `src/pages/api/` | Serverless functions | newsletter, contact, reactions, OG image, metrics |
| `src/layouts/` | Page shell | Single `Layout.astro` — props-driven SEO/meta, ClientRouter, theme |
| `src/utils/` | Shared logic | OG generation, SEO helpers, GitHub API, newsletter, Sentry, schema.org |
| `src/middleware/` | Request middleware | Security headers (`security.ts`), OG image caching (`og-cache.ts`) |
| `src/styles/` | Styling | `global.css` (2381 lines, Tailwind v4 + 7 themes), critical/ CSS split by route |
| `src/content/` | Blog content | MDX frontmatter with schema-validated collections (`content.config.ts`) |
| `src/types/` | TypeScript definitions | Schema.org JSON-LD types (`schema.ts`) |
| `scripts/` | Build pipeline | 20+ scripts: build orchestration, RSS, sitemap, critical CSS, image migration |
| `tests/` | Playwright E2E | 17+ spec files covering all routes + OG + a11y + performance |
| `public/` | Static assets | Service worker, fonts, icons, generated RSS/sitemap/PDF |

---

## Development Commands

```bash
# Development
bun run dev              # Start dev server (port 4321)
bun run start            # Alias for dev

# Build
bun run build            # Full pipeline (typegen → images → astro → pagefind → sitemap → RSS)
bun run preview          # Preview production build

# Lint & Format (Biome)
bun run lint             # Lint src/
bun run lint:fix         # Lint with auto-fix
bun run format           # Format all files with Biome
bun run check            # Full Biome check (lint + format + organize imports)
bun run check:write      # Check with auto-fix
bun run ci               # CI check (read-only, no writes)

# Testing (Playwright)
bun run test             # Run all Playwright tests
bun run test:smoke       # Smoke tests only
bun run test:headed      # Headed browser mode
bun run test:ui          # Playwright UI mode
bun run test:report      # Show HTML report
node run-og-tests.js     # OG-specific test suite


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piyush97/PiyushMehta.com](https://github.com/piyush97/PiyushMehta.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

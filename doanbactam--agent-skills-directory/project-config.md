---
trigger: always_on
description: **AGNXI** is an agent skills directory for coding assistants. Users discover SKILL.md workflows, compare stars/updates, and install skills for Claude Code, Cursor, Windsurf, Amp, and more.
---

# AGENTS.md - Project Rules for AI Agents

## Overview

**AGNXI** is an agent skills directory for coding assistants. Users discover SKILL.md workflows, compare stars/updates, and install skills for Claude Code, Cursor, Windsurf, Amp, and more.

### Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js (App Router, Turbopack) | 16.1.6 |
| UI | React | 19.2.4 |
| Language | TypeScript (strict mode) | 5.9 |
| Styling | Tailwind CSS v4 (CSS-first) + shadcn | 4.2 |
| CSS Tooling | PostCSS with `@tailwindcss/postcss` | — |
| UI Primitives | Base UI (`@base-ui/react`) | 1.1 |
| Variants | class-variance-authority (CVA) | 0.7 |
| Icons | lucide-react | 0.575 |
| Auth | Clerk (`@clerk/nextjs`) | 6.38 |
| Database | PostgreSQL via Drizzle ORM | 0.45 |
| AI | Vercel AI SDK + Google Gemini (`@ai-sdk/google`) | 6.0 |
| Background Jobs | Inngest | 3.52 |
| Rate Limiting | Upstash Redis (`@upstash/ratelimit`) | 2.0 |
| URL State | nuqs | 2.8 |
| Env Validation | `@t3-oss/env-nextjs` + Zod v4 | — |
| Toasts | Sonner | 2.0 |
| Markdown | react-markdown + remark-gfm | — |
| Analytics | `@next/third-parties` | 16.1 |
| Tables | `@tanstack/react-table` | 8.21 |
| Runtime | Bun (package manager + runtime) | — |
| Node | ≥ 20.0.0 | — |

---

## Commands

**ALWAYS use Bun instead of npm/yarn/pnpm.**

```bash
# Development
bun dev                    # Start dev server (port 4000, Turbopack)

# Build & Production
bun build                  # Production build
bun start                  # Start production server (port 4000)

# Linting
bun lint                   # Run ESLint (always run after code changes)

# Database (Drizzle ORM)
bun db:generate            # Generate Drizzle migrations
bun db:migrate             # Run Drizzle migrations
bun db:push                # Push schema directly to database
bun db:studio              # Open Drizzle Studio (visual DB browser)
bun db:seed:categories     # Seed category data

# Tests
# No test framework is currently configured.
# Ask the user before adding any testing setup.
```

**Important notes:**
- Always run `bun lint` after significant code changes
- Dev server runs on port **4000** (not default 3000)
- Turbopack is the default bundler in dev mode
- The `postinstall` script auto-generates Drizzle migrations

---

## Project Structure

```
app/                      # Next.js App Router
├── (auth)/               # Auth routes (sign-in, sign-up)
├── [owner]/              # Dynamic owner pages
├── about/                # About page
├── admin/                # Admin dashboard (role-protected)
├── agent-skills/         # Agent skills listing
├── api/                  # API routes
│   ├── admin/            # Admin API endpoints
│   ├── inngest/          # Inngest webhook handler
│   ├── skills/           # Skills CRUD API
│   ├── webhooks/         # GitHub webhooks
│   ├── health/           # Health check
│   ├── stats/            # Stats endpoint
│   ├── owners/           # Owner data
│   └── categories/       # Category data
├── categories/           # Category pages
├── skills/               # Skill detail pages
├── globals.css           # Tailwind v4 theme (oklch tokens, shadcn vars)
├── layout.tsx            # Root layout (fonts, Clerk, analytics)
├── page.tsx              # Homepage
├── sitemap.ts            # Dynamic sitemap generation
├── error.tsx             # Error boundary
└── not-found.tsx         # 404 page

components/               # Shared React components
├── ui/                   # Reusable UI primitives (shadcn-style)
│   ├── button.tsx        # Button with CVA variants
│   ├── dialog.tsx        # Base UI Dialog wrapper
│   ├── select.tsx        # Base UI Select wrapper
│   ├── dropdown-menu.tsx # Base UI Menu wrapper
│   ├── input.tsx         # Input component
│   ├── card.tsx          # Card layout
│   ├── badge.tsx         # Badge with variants
│   ├── tooltip.tsx       # Tooltip component
│   ├── data-table.tsx    # TanStack Table wrapper
│   ├── sheet.tsx         # Slide-out panel
│   ├── sonner.tsx        # Toast provider
│   └── ...               # 23 UI components total
├── layouts/              # Page layout components
├── seo/                  # SEO components (JSON-LD, meta)
├── analytics/            # Analytics integration
├── faq/                  # FAQ components
└── error-boundary.tsx    # Client error boundary

features/                 # Feature modules (domain logic)
├── skills/               # Skills grid, filters, search
├── submissions/          # Skill submission flow
└── marketing/            # Landing page components

lib/                      # Shared utilities and logic
├── db/                   # Database layer
│   ├── schema.ts         # Drizzle schema (skills, categories, syncJobs, etc.)
│   ├── queries.ts        # Database queries
│   ├── client.ts         # Drizzle client setup
│   ├── transaction.ts    # Transaction helpers
│   ├── errors.ts         # Database error handling
│   └── seed-categories.ts
├── actions/              # Server Actions
├── ai/                   # AI integration (Gemini)
├── inngest/              # Inngest functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doanbactam/agent-skills-directory](https://github.com/doanbactam/agent-skills-directory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

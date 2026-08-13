---
trigger: always_on
description: An open-source, self-hosted invoicing dashboard. Lightweight, modern, and designed for deployment on Dokploy or Coolify.
---

# CLAUDE.md — Autonomous Development Guide

## Project: Inkvoice (Open Source)

An open-source, self-hosted invoicing dashboard. Lightweight, modern, and designed for deployment on Dokploy or Coolify.

---

## Tech Stack

### Runtime & Backend
- **Runtime**: Bun (latest stable)
- **Framework**: Hono v4 — ultra-lightweight (~14KB), fast, TypeScript-first
- **Database**: SQLite via `bun:sqlite` (native zero-copy bindings, no external dependency)
- **Auth**: JWT via `hono/jwt` + bcrypt for password hashing
- **Validation**: Zod for request/response validation

### Frontend
- **Framework**: React 19 + Vite 6
- **Routing**: React Router v7 (SPA mode)
- **Styling**: Tailwind CSS v4 + shadcn/ui components
- **State**: Zustand (lightweight, ~1KB)
- **Icons**: Lucide React
- **Charts**: Recharts (lightweight charting)
- **HTTP Client**: Built-in fetch with a typed API client wrapper

### PDF Generation
- **Primary**: Puppeteer with Chrome Headless Shell
- **Templates**: Mustache (handlebars-compatible, logic-less templates)

### Build & Deploy
- **Monorepo**: Single repo, `packages/backend` + `packages/frontend`
- **Build**: Vite builds frontend to static files; Hono serves both API + static
- **Docker**: Single Dockerfile, single container (Bun serves everything)
- **Target RAM**: 50-100MB base, 200-400MB peak during PDF generation

### Why This Stack (vs Next.js)
- **Next.js**: 200-400MB base RAM, heavy for self-hosted. Overkill for a dashboard.
- **Hono + React SPA on Bun**: 50-100MB base RAM. Bun's native SQLite is zero-copy. Hono is 14KB. Static React build adds 0 server RAM.
- **Single container**: Hono serves the API at `/api/*` and the built React SPA for everything else. No nginx, no reverse proxy complexity.
- **Result**: 2-4x lighter than Next.js while maintaining modern React DX.

---

## Project Structure

```
inkvoice/
├── CLAUDE.md                    # This file
├── package.json                 # Root workspace config
├── bunfig.toml                  # Bun configuration
├── Dockerfile                   # Single-stage production build
├── docker-compose.yml           # Production compose
├── docker-compose.dev.yml       # Development compose
├── .env.example                 # Environment variable template
├── .gitignore
├── LICENSE                      # MIT License
├── README.md                    # User-facing documentation
│
├── packages/
│   ├── backend/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── src/
│   │       ├── index.ts              # Entry point — Hono app + static serving
│   │       ├── app.ts                # Hono app setup (middleware, routes)
│   │       ├── database/
│   │       │   ├── connection.ts      # SQLite connection singleton
│   │       │   ├── migrations.ts      # Schema migrations (versioned)
│   │       │   └── seed.ts            # Default data seeding
│   │       ├── routes/
│   │       │   ├── auth.ts            # POST /api/v1/auth/login, /logout
│   │       │   ├── invoices.ts        # CRUD + publish/void/duplicate/pdf
│   │       │   ├── customers.ts       # CRUD
│   │       │   ├── products.ts        # CRUD + categories + units
│   │       │   ├── templates.ts       # CRUD + install + preview
│   │       │   ├── settings.ts        # GET/PUT business settings
│   │       │   ├── tax.ts             # Tax definitions CRUD
│   │       │   ├── users.ts           # User management + permissions
│   │       │   ├── dashboard.ts       # Dashboard stats + chart data
│   │       │   └── public.ts          # Public invoice view + PDF download
│   │       ├── middleware/
│   │       │   ├── auth.ts            # JWT verification + permission check
│   │       │   ├── rate-limiter.ts    # Login brute-force protection
│   │       │   ├── security.ts        # Security headers (CSP, HSTS, etc.)
│   │       │   └── error-handler.ts   # Global error handling
│   │       ├── services/
│   │       │   ├── invoice.service.ts  # Invoice business logic
│   │       │   ├── customer.service.ts
│   │       │   ├── product.service.ts
│   │       │   ├── template.service.ts
│   │       │   ├── auth.service.ts
│   │       │   ├── settings.service.ts
│   │       │   ├── tax.service.ts
│   │       │   ├── user.service.ts
│   │       │   ├── pdf.service.ts      # HTML-to-PDF via Chrome Headless
│   │       │   └── dashboard.service.ts
│   │       ├── utils/
│   │       │   ├── jwt.ts
│   │       │   ├── password.ts
│   │       │   ├── invoice-number.ts  # Number pattern generation
│   │       │   ├── tax-calculator.ts  # Tax computation logic
│   │       │   ├── currency.ts        # Currency formatting
│   │       │   └── env.ts             # Env config with defaults
│   │       └── types/
│   │           ├── invoice.ts
│   │           ├── customer.ts
│   │           ├── product.ts
│   │           ├── settings.ts
│   │           ├── user.ts
│   │           └── common.ts
│   │
│   └── frontend/
│       ├── package.json
│       ├── tsconfig.json
│       ├── vite.config.ts
│       ├── index.html
│       ├── tailwind.config.ts
│       └── src/
│           ├── main.tsx               # React entry point
│           ├── App.tsx                # Router setup
│           ├── api/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pigontech/inkvoice](https://github.com/pigontech/inkvoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

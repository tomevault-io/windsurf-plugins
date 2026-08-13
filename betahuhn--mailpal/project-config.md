---
trigger: always_on
description: Generates random alias slugs in the format `adjective-noun-number` using hard-coded word lists. Used when creating aliases without a custom local part.
---

# MailPal — Copilot Coding Agent Instructions

## Project Overview

MailPal is a **self-hosted email alias forwarding dashboard** built entirely on Cloudflare's free tier (Pages + Workers + KV). It lets users create and manage email aliases (e.g. `swift-meadow-412@yourdomain.com`) that forward to a real inbox, with per-alias enable/disable controls, wildcard mode, stats tracking, multi-domain support, and optional password protection.

There are two deployable units that share a single KV namespace:
- **SvelteKit app** (`/`) — management UI + REST API, deployed to Cloudflare Pages
- **Email worker** (`email-worker/`) — intercepts incoming mail via Cloudflare Email Routing and forwards or rejects based on KV state, deployed as a Cloudflare Worker

---

## Technology Stack

| Layer | Technology |
|---|---|
| UI framework | Svelte 5 + SvelteKit 2 |
| Styling | Tailwind CSS 3 (custom dark theme) + PostCSS + Autoprefixer |
| UI components | bits-ui 2 (headless Svelte components) |
| Build tool | Vite 5 |
| Type safety | TypeScript 5 (strict mode) |
| Deployment target | Cloudflare Pages (frontend) + Cloudflare Workers (email worker) |
| Storage | Cloudflare KV |
| Infra CLI | Wrangler 4 |
| CF types | @cloudflare/workers-types 4 |

There is **no traditional database, no persistent Node.js server in production, and no test framework** — the production runtime runs entirely on Cloudflare's edge (Node.js is used only for local development and build tooling).

---

## Repository Structure

```
MailPal/
├── src/                         # SvelteKit application
│   ├── app.css                  # Global CSS (Tailwind base)
│   ├── app.d.ts                 # Global TS types (KV binding on platform.env)
│   ├── app.html                 # HTML template
│   ├── hooks.server.ts          # Auth middleware — runs before every request
│   ├── lib/
│   │   ├── auth.ts              # HMAC session cookie sign/verify helpers
│   │   ├── kv.ts                # KV namespace read/write helpers
│   │   ├── sluggen.ts           # Random alias slug generator (adjective-noun-number)
│   │   ├── types.ts             # Shared TypeScript interfaces (DomainConfig, AliasConfig, etc.)
│   │   └── components/          # 11 Svelte UI components (dialogs, forms, sidebar, etc.)
│   └── routes/
│       ├── +layout.server.ts    # Layout server (passes auth state to all pages)
│       ├── +layout.svelte       # App shell
│       ├── +page.server.ts      # Home: loads domains, aliases, destinations from KV
│       ├── +page.svelte         # Main dashboard UI (~1700 lines)
│       ├── login/               # Login form + action
│       ├── logout/              # Logout endpoint (clears cookie)
│       ├── domains/[domain]/    # Domain sub-page (redirects to home with domain selected)
│       └── api/                 # REST API endpoints (see API section below)
├── email-worker/
│   ├── src/index.ts             # Cloudflare Email Worker (email forwarding logic)
│   ├── package.json
│   ├── tsconfig.json
│   └── wrangler.toml            # Wrangler config for the email worker
├── package.json                 # Main project scripts and dependencies
├── svelte.config.js             # SvelteKit config (adapter-cloudflare)
├── vite.config.ts               # Vite config (sveltekit plugin)
├── tailwind.config.ts           # Tailwind theme (custom `app.*` color tokens)
├── tsconfig.json                # TS config (extends .svelte-kit/tsconfig.json)
├── wrangler.toml                # Wrangler config for Cloudflare Pages
└── postcss.config.js
```

---

## Build, Lint, and Dev Commands

### Main project (SvelteKit app)

```bash
npm install                  # Install dependencies
npm run dev                  # Start Vite dev server at http://localhost:5173
npm run build                # Build for Cloudflare Pages (.svelte-kit/cloudflare/)
npm run check                # svelte-check — type-check all Svelte + TS files (primary linter)
npm run check:watch          # Same, in watch mode
npm run deploy:frontend      # build + wrangler pages deploy
npm run deploy               # deploy:frontend + deploy:worker
```

### Email worker

```bash
cd email-worker
npm install
npm run dev                  # wrangler dev (local email worker simulation)
npm run deploy               # wrangler deploy (deploy to Cloudflare)
```

### No test suite

There are **no automated tests** in this repository (no Vitest, Jest, etc.). Validation is done via:
- `npm run check` — svelte-check (type errors + Svelte-specific warnings)
- `npm run build` — catches TypeScript compilation errors
- Manual testing against a real or local Cloudflare environment

When making changes, always run `npm run check` before committing. The build step is also a reliable gate.

---

## Key Source Files and Patterns

### Data types (`src/lib/types.ts`)

```typescript
interface DomainConfig {
  domain: string
  targetEmail: string        // Default forwarding destination
  wildcardEnabled: boolean   // Auto-create aliases on first email received
  enabled: boolean
  createdAt: number
  color?: string             // Optional hex color for UI sidebar
}

interface AliasConfig {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BetaHuhn/MailPal](https://github.com/BetaHuhn/MailPal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->

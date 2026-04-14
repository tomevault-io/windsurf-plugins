---
trigger: always_on
description: <!-- Last verified: 2026-03-16 -->
---

<!-- Last verified: 2026-03-16 -->
# GEMINI.md — Gemini CLI / Gemini Agent Context

> Context file for Google Gemini CLI (`gemini` CLI tool) and Gemini-powered coding agents.
> Keep this file updated when architecture, conventions, or commands change.

---

## Project Summary

**Interact** is an enterprise-grade employee engagement and gamification platform (v0.0.0, active development).
It is a React 18 + Vite 6 single-page application backed by Base44 SDK serverless functions running on Deno.
The platform delivers points, badges, leaderboards, AI-powered activity recommendations, and team analytics
for organizations improving employee retention and cohesion.

**Repository:** `Krosebrook/interact`
**Canonical docs:** `docs/README.md` → all documentation navigation
**Architecture:** See [ARCHITECTURE.md](./docs/ARCHITECTURE.md) and [docs/adr/](./docs/adr/)

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| UI Framework | React | 18.2.0 |
| Build Tool | Vite | 6.1.0 |
| Language | JavaScript (JSX) — TypeScript migration Q2 2026 | — |
| Styling | TailwindCSS + Radix UI primitives | 3.4.17 |
| State (server) | TanStack Query | 5.84.1 |
| State (UI) | React Context API | — |
| Forms | React Hook Form + Zod | 7.54.2 / 3.24.2 |
| Animations | Framer Motion | 11.16.4 |
| Backend | Base44 SDK (Deno serverless functions) | 0.8.20 (frontend) / 0.8.6 (Deno) |
| Auth | Base44 Auth (JWT) | — |
| Testing | Vitest + React Testing Library | 4.0.17 |
| CI/CD | GitHub Actions → Vercel | — |
| Node.js | 20 (LTS) | >=20 required |

---

## Key Directories

```
src/
  App.jsx               — Router setup, provider composition
  Layout.jsx            — App shell (nav, sidebar, auth guard)
  main.jsx              — Vite entry point
  pages.config.js       — Auto-generated static page map (127 pages)
  pages/                — 127 route-level page components
  components/           — 64 feature component sub-directories
    ui/                 — Radix UI / shadcn primitives
    common/             — Shared utility components
    gamification/       — Points, badges, leaderboards
    analytics/          — Charts and dashboards
    ai/                 — AI-powered components
    docs/               — Auto-generated in-app docs (EXCLUDED from lint)
  api/
    base44Client.js     — Base44 SDK client (uses VITE_BASE44_APP_ID, VITE_BASE44_BACKEND_URL)
  lib/
    AuthContext.jsx     — useAuth hook and auth context
    query-client.js     — TanStack Query client config (staleTime: 5 min)
    utils.js            — cn() helper for Tailwind class merging
    imageUtils.js       — Cloudinary URL optimization
    app-params.js       — Env param resolution
  hooks/                — Custom React hooks (prefix: use*)
  utils/                — Pure utility functions
  test/                 — Test setup, fixtures, and unit tests
functions/              — ~200 Deno/TypeScript serverless functions
docs/                   — Structured documentation (architecture, ADRs, guides)
ADR/                    — Root-level ADR copies (canonical: docs/adr/)
scripts/                — CLI and tooling scripts
.github/workflows/      — ci.yml, docs-authority.yml, safe-merge-checks.yml
```

---

## Environment Variables

### Frontend (Vite `VITE_` prefix — bundled into browser build)

| Variable | Required | Description |
|---|---|---|
| `VITE_BASE44_APP_ID` | ✅ | Base44 application identifier |
| `VITE_BASE44_BACKEND_URL` | ✅ | Base44 backend URL (e.g. `https://api.base44.app`) |
| `VITE_COMPANY_ID` | Optional | Multi-tenant company identifier |
| `VITE_ENABLE_ANALYTICS` | Optional | Feature flag — analytics (`true`/`false`) |
| `VITE_ENABLE_PWA` | Optional | Feature flag — PWA mode (`true`/`false`) |
| `VITE_GOOGLE_ANALYTICS_ID` | Optional | Google Analytics GA4 ID |
| `VITE_STRIPE_PUBLISHABLE_KEY` | Optional | Stripe public key (payments) |
| `VITE_SENTRY_DSN` | Optional | Sentry error tracking DSN |

**Rule:** Never put secrets in `VITE_` variables — they are exposed to the browser.

### Backend (Deno functions — set in Base44 dashboard)

| Variable | Required | Description |
|---|---|---|
| `OPENAI_API_KEY` | ✅ (AI features) | OpenAI GPT-4 |
| `ANTHROPIC_API_KEY` | ✅ (AI features) | Anthropic Claude 3 |
| `GOOGLE_API_KEY` | Optional | Gemini Pro |
| `SLACK_BOT_TOKEN` | Optional | Slack integration |
| `STRIPE_SECRET_KEY` | Optional | Stripe payments |
| `TWILIO_ACCOUNT_SID` | Optional | SMS via Twilio |
| `CLOUDINARY_CLOUD_NAME` | Optional | Media uploads |
| `SENDGRID_API_KEY` | Optional | Email notifications |

---

## Build & Run Commands

```bash
# Install dependencies (CI-safe, exact versions)
npm ci

# Development server
npm run dev                  # http://localhost:5173

# Production build
npm run build

# Preview production build
npm run preview

# Lint (must pass before commit)
npm run lint                 # check
npm run lint:fix             # auto-fix

# Tests
npm run test:run             # single pass (CI mode)
npm test                     # watch mode
npm run test:coverage        # with coverage report

# Type check
npm run typecheck

# Generate PRD
npm run prd:generate
```

---

## Code Conventions

### Component Structure

```javascript
// Order: imports → constants → hooks (TOP LEVEL ONLY) → handlers → JSX

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Krosebrook) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

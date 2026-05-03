---
trigger: always_on
description: An AI-guided writing tool that structures your thinking without doing the writing for you. Built on the Dignified Technology design philosophy. React 19, Supabase, Express 5, Anthropic Claude.
---

# Hermes

An AI-guided writing tool that structures your thinking without doing the writing for you. Built on the Dignified Technology design philosophy. React 19, Supabase, Express 5, Anthropic Claude.

## Open Source — Security Rules

This is an **open-source repository**. Every file, commit, and PR is publicly visible. Follow these rules strictly:

- **Never commit secrets**: No API keys, tokens, passwords, DSNs, or credentials in code or config files. All secrets go in `.env` files (which are `.gitignore`d).
- **Never hardcode URLs with credentials**: No Supabase service keys, Sentry DSNs, or third-party tokens inline.
- **Audit before committing**: Before staging files, verify no `.env`, credentials, or private keys are included. If in doubt, ask.
- **Plans and PR descriptions**: Do not include real API keys, passwords, or internal URLs. Use placeholders like `YOUR_API_KEY` or `<redacted>`.
- **Review diffs carefully**: Check `git diff` output for accidental secret leaks before every commit.
- **Environment-specific values**: Always reference env vars (`process.env.X`, `import.meta.env.VITE_X`) — never inline the actual values.

## Quick Start

```bash
# Both frontend (port 5176) + backend (port 3003)
npm run dev

# Or separately:
npm run web:dev      # Frontend only
npm run server:dev   # Backend only
```

## Architecture

**Frontend**: React 19 + Vite 7 + react-router-dom + CSS Modules
**Backend**: Express 5 + Anthropic Claude (`/server/src/`)
**Database**: Supabase (PostgreSQL + Auth)

### Provider hierarchy

```
Sentry.ErrorBoundary → BrowserRouter → AuthProvider → App
```

### Route structure

```
/                       → RedirectToLatestProject (redirects to /projects/:id)
/projects/:projectId    → FocusPage (auth required)
/login                  → LoginPage (standalone login form)
/signup                 → SignupPage (standalone signup form)
/forgot-password        → Redirect to / (forgot password lives in UserMenu dropdown)
/reset-password         → ResetPasswordPage
/auth/confirm           → AuthConfirmPage
*                       → NotFound (404)
```

## Project Structure

```
apps/web/src/
  pages/
    FocusPage/              # Main writing workspace (AI assistant + editor)
  components/
    MarkdownText/           # Markdown rendering component
  contexts/
    AuthContext.jsx          # Auth state management
  hooks/                    # useAuth + data fetching hooks
  styles/                   # Shared CSS primitives (form, dropdown)

packages/
  api/src/writing.ts        # TypeScript interfaces + client API functions
  domain/                   # Shared pure domain utils (relativeTime)

server/src/
  index.ts                  # Express entry (port 3001)
  routes/assistant.ts       # Assistant chat endpoint (SSE streaming with highlights)
  lib/                      # supabase.ts, logger.ts (pino)
  middleware/auth.ts        # JWT verification
```

## API endpoints

**Implemented** (`server/src/routes/assistant.ts`):

- `POST /api/assistant/chat` — contextual assistant chat with inline highlights (SSE)

**Auth** (`server/src/routes/auth.ts`):

- `POST /api/auth/signup` — create user account (email/password, auto-confirmed); stamps `trial_expires_at` (defaults to `FREE_TIER_DAYS`)

**Billing** (`server/src/routes/stripe.ts` + `server/src/routes/usage.ts`):

- `POST /api/stripe/webhook` — Stripe webhook handler (signature-verified, idempotent)
- `POST /api/stripe/portal` — create Stripe Customer Portal session (auth required)
- `GET /api/usage/current` — current user's message usage and plan info (auth required)

### Database tables

All tables linked by `project_id`, owner-scoped via RLS:

- `projects` — `id`, `user_id`, `title`, `status`, `content`, `highlights` (JSONB), timestamps
- `assistant_conversations` — `project_id` (unique), `messages` (JSONB), timestamps
- `user_profiles` — `id` (PK → auth.users), `plan`, `stripe_customer_id`, `stripe_subscription_id`, `subscription_status`, `billing_cycle_anchor`, `cancel_at_period_end`, `current_period_end`, `trial_expires_at`, timestamps
- `message_usage` — `id`, `user_id`, `project_id`, `created_at` (tracks per-message usage for limits)
- `processed_stripe_events` — `event_id` (PK), `event_type`, `processed_at` (webhook idempotency)

## Staging Environment

### Deployed staging

- **Frontend**: `https://staging.dearhermes.com` (Vercel, aliased via CI on every PR)
- **Backend**: Railway `staging` environment (deploys from PR via CI)
- **Supabase**: Separate `hermes-staging` project (us-east-1)

### How staging deploys work

1. Open a PR against `main` → CI runs 5 checks in parallel
2. All checks pass → `deploy-staging` job deploys frontend to Vercel (aliased to `staging.dearhermes.com`) AND server to Railway staging via `railway deployment up`
3. Test on `staging.dearhermes.com` → merge PR → production auto-deploys from `main`

### Running locally against staging

```bash
# Frontend (port 5176, staging Supabase)
npm run web:dev:staging

# Backend (port 3003, staging Supabase) — separate terminal
npm run server:dev:staging
```

### How it works


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Egotistical-Engineering/hermes](https://github.com/Egotistical-Engineering/hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

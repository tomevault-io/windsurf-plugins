---
trigger: always_on
description: This repository contains **SquadRidge**, a pilot‑stage verified dialogue infrastructure for structured, facilitator‑led cross‑border dialogue.
---

# GitHub Copilot instructions for this repository

## What this repo is

This repository contains **SquadRidge**, a pilot‑stage verified dialogue infrastructure for structured, facilitator‑led cross‑border dialogue.

SquadRidge is **not** a generic chat app.
It focuses on:
- Verified access (pilot‑scoped identity / attribute proofs, Semaphore‑style).
- Pseudonymous small‑squad matching and structured session rooms.
- Facilitator and moderator affordances, including safety workflows.
- Encrypted‑at‑rest session chat and RLS‑enforced access.
- Public, anonymous, timestamped ledger‑style outcome records for squad consensus.

Impact claims are intentionally modest: this repo supports *serious bounded pilots*, not “solved global conflict prevention” or fully operator‑blind, public early‑warning infrastructure.

## Tech stack and architecture

- Frontend: React + Vite + TypeScript.
- Styling: Tailwind CSS.
- Backend: pure BaaS via Supabase (PostgreSQL + RLS, Auth, Realtime, Edge Functions).
- No Node server tier and **no local Redis** dependency.
- Edge rate limiting (when enabled) uses Upstash Redis via REST from `supabase/functions/rate-limit/`.

When older docs mention `docker-compose.yml` or a local Redis stub, treat them as obsolete; follow the current Supabase‑only architecture.

## Environment and configuration

When generating or modifying code that relies on environment variables, assume:

- `VITE_SUPABASE_URL` — Supabase project URL (`https://<ref>.supabase.co`).
- `VITE_SUPABASE_PUBLISHABLE_KEY` (preferred) or legacy `VITE_SUPABASE_ANON_KEY`.
- `VITE_SITE_URL` — deployed origin (used for magic links).
- Optional flags such as `VITE_ENABLE_DEMO_SQUAD` and any other documented `VITE_*` keys in `.env.example`.

Guidelines for env usage:

- Never hard‑code secrets, keys, or project refs.
- Use `import.meta.env.VITE_*` on the client.
- Assume `.env.example` documents the intended shape; keep new envs documented there.
- `.env*` files must **never** be committed; they are gitignored.

## How to run, build, and test

When adding instructions or scripts, align with the existing script surface:

- `npm run dev` — Vite dev server for local development.
- `npm run build` — typecheck + production build, also writes `dist/partner-one-pager.html`.
- `npm run build:e2e` — build for Playwright / demo smoke using `.env.e2e`.
- `npm run preview` — preview production build.
- `npm run e2e` — hermetic Playwright suite (default project).
- `npm run e2e:staging` — staging‑only specs (`*-staging.spec.ts`).
- `npm run demo:capture` — scripted tour screenshots under `test-results/demo-capture/`.
- `npm run gen:partner-one-pager` — regenerate the printable partner one‑pager from `docs/business/pilot-partner-one-pager.md`.

When adding new tests, prefer Playwright for E2E flows that touch auth/RLS/Realtime and align with the existing `e2e` and `e2e:staging` project patterns.

## Supabase, migrations, and CI

This repo assumes a Supabase project is linked and migrations are applied via CI:

- Supabase resources (tables, RLS, auth, Realtime, Edge Functions) live under `supabase/`.
- Migrations are the source of truth for schema.
- After linking Supabase + GitHub, the `Deploy Supabase to production` workflow on `main` should succeed.

The Supabase deploy workflow (`.github/workflows/deploy-supabase-production.yml`):

- Uses repository secrets:
    - `SUPABASE_ACCESS_TOKEN` — classic PAT (`sbp_*`), *not* anon or service_role.
    - `SUPABASE_DB_PASSWORD` — database password for migrations.
    - `SUPABASE_PROJECT_ID` — 20‑character project ref (e.g. the `<ref>` in `https://<ref>.supabase.co`).
- Validates token and ref shape before running.
- Deploys database migrations only; static frontend is shipped via a separate pipeline.

When Copilot edits CI YAML:

- Preserve existing validations and comments, especially around PAT formats.
- Do not introduce new secrets without also updating documentation.
- Keep deploy steps idempotent and safe to re‑run.

## Frontend hosting

The static app is deployed separately from Supabase migrations:

- Use the same `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY` (or anon), and `VITE_SITE_URL` as in local production builds.
- Set env vars in the hosting provider (Vercel, Netlify, Cloudflare Pages, etc.), not just in local `.env`.
- `.github/workflows/deploy-frontend.yml`:
    - Runs tests and `npm run build` (with `VITE_ZK_STUB=false`).
    - Uploads `dist/` as a build artifact for download / attachment to a host if desired.

When Copilot touches build or deploy logic, keep these invariants:

- Builds must respect feature flags like `VITE_ZK_STUB` and `VITE_ENABLE_DEMO_SQUAD`.
- The default pipeline assumes real ZK / security‑relevant code is *on* in production; stubs are for demos and development only.
- Do not bypass tests or readiness checks without an explicit, documented reason.

## Domain concepts and flows

Key flows the codebase supports today:

1. **Verify**
    - Pilot‑scoped access and attribute verification (Semaphore‑style).
    - Treat ZK‑ish verification as a first‑class concern; keep flows composable for future cryptographic upgrades.

2. **Convene**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brass4698-coder/squadridge-v1](https://github.com/brass4698-coder/squadridge-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->

---
trigger: always_on
description: Accounted is a Swedish-focused accounting SaaS for sole traders (enskild firma) and limited companies (aktiebolag). It implements double-entry bookkeeping compliant with Swedish accounting law (Bokföringslagen), including VAT handling, tax reporting, and 7-year document retention. Multi-tenant: each user can own or be a member of multiple companies, optionally grouped into teams (for consultants).
---

# CLAUDE.md — Accounted

## Project Overview

Accounted is a Swedish-focused accounting SaaS for sole traders (enskild firma) and limited companies (aktiebolag). It implements double-entry bookkeeping compliant with Swedish accounting law (Bokföringslagen), including VAT handling, tax reporting, and 7-year document retention. Multi-tenant: each user can own or be a member of multiple companies, optionally grouped into teams (for consultants).

**Tech stack**: Next.js 16.1.5 (App Router), React 19.2.3, TypeScript 5 (strict), Zod 4, Supabase (PostgreSQL + RLS + email/password + TOTP MFA auth), Tailwind CSS 4 + shadcn/ui, Vercel hosting, Docker (self-hosted).

**Integrations**: Enable Banking (PSD2), TIC Identity, Anthropic SDK, AWS Bedrock, OpenAI, Resend, Sentry, Svix, web-push, Upstash Redis, Google Drive, JSZip, sharp, Framer Motion, Recharts, PDF.js, `@react-pdf/renderer`, xlsx, fuse.js, ics.

**Path alias**: `@/*` maps to the project root. **Language**: All code, comments, and commit messages in English. **License**: AGPL-3.0-or-later.

---

## Commands

```bash
npm run dev              # Start dev server (runs setup:extensions first)
npm run build            # Production build (runs setup:extensions first)
npm run lint             # ESLint
npm test                 # Run all Vitest tests
npx vitest run <dir>     # Run tests in a specific directory
npm run test:pg          # pg-real tests against real Postgres
npm run setup:extensions # Regenerate extension registry from extensions.config.json
npm run skills:generate  # Regenerate agent_atom_registry seed migration after editing an atom SKILL.md
npm run skills:check     # CI guard: fail if an atom SKILL.md changed without regenerating the seed migration
```

---

## Key Architectural Relationships

- **Multi-tenant model**: `companies` owns all business data. `company_members` links users to companies (owner/admin/member/viewer). `teams` group companies. Context resolved via `gnubok-company-id` cookie in `lib/supabase/middleware.ts`.
- **All journal entry creation** routes through `lib/bookkeeping/engine.ts`. Lifecycle: `createDraftEntry()` → `commitEntry()` (atomic voucher via `commit_journal_entry` RPC). `createJournalEntry()` does both. Reversal: `reverseEntry()`. Correction: `correctEntry()` in `lib/core/bookkeeping/storno-service.ts`.
- **API routes** emitting events must call `ensureInitialized()` (`lib/init.ts`) at module level to load extensions and wire handlers.
- **Event bus** (`lib/events/bus.ts`) is a module-level singleton using `Promise.allSettled`. 50+ event types in `lib/events/types.ts`. Persisted to `event_log` table (30-day TTL).
- **Supabase clients**: browser (`client.ts`), server cookies (`createClient()`), service role (`createServiceClient()`), cookieless service role for API keys (`createServiceClientNoCookies()`). Pagination: `fetchAllRows()`.
- **Extension system**: Opt-in via `extensions.config.json`. Core runs with zero extensions.
- **Types**: Shared types in `types/index.ts` (~3,100 lines). Import via `import type { T } from '@/types'`. Event types in `lib/events/types.ts`. Extension types in `lib/extensions/types.ts`.
- **Error messages**: `lib/errors/get-error-message.ts` maps to Swedish (Zod → Postgres → HTTP → fallback).

---

## Repository Map

- `lib/bookkeeping/` — Engine, entry generators, mapping, templates, BAS data
- `lib/core/` — Period, year-end, storno, tax codes, audit, documents
- `lib/events/` — Bus singleton, event types, event log handler
- `lib/auth/` — API keys, require-auth/write, MFA, OAuth codes, invite tokens, cron, BankID
- `lib/supabase/` — Clients, middleware, `fetchAllRows` pagination
- `lib/api/` — Zod validation (`validateBody`/`validateQuery`), schemas
- `lib/reports/` — Report generators (balance sheet, income statement, trial balance, GL, AR/supplier ledger, VAT declaration, SIE, INK2, NE-bilaga, KPI, salary, vacation, …)
- `lib/invoices/`, `lib/transactions/`, `lib/import/` (SIE/bank/opening balance), `lib/documents/` (matchers)
- `lib/providers/` — Fortnox, Bokio, Briox, BL, Visma (OAuth, retry, consent)
- `lib/salary/` — Payroll engine, tax tables, AGI, KU, payslips, löneväxling, personnummer
- `lib/reconciliation/`, `lib/tax/`, `lib/vat/` (VIES, MOMS box), `lib/deadlines/`, `lib/currency/` (Riksbanken), `lib/skatteverket/`, `lib/bankgiro/` (Luhn), `lib/calendar/` (ICS)
- `lib/utils.ts` (`cn()`, `formatCurrency()`, `formatDate()`, `formatOrgNumber()`), `lib/logger.ts`
- `app/(dashboard)/*` — pages; `app/api/*` — API routes; `supabase/migrations/` — schema; `extensions/general/*` — opt-in extensions
- Path-scoped detail lives in `.claude/rules/` (see **Path-scoped rules** below).

---

## Multi-Tenant Architecture

- **companies**: Business unit. All business data has a `company_id` column.
- **company_members**: Roles `owner`/`admin`/`member`/`viewer`, source `direct`|`team`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erp-mafia/accounted](https://github.com/erp-mafia/accounted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

---
trigger: always_on
description: gnubok is a Swedish-focused accounting SaaS for sole traders (enskild firma) and limited companies (aktiebolag). It implements double-entry bookkeeping compliant with Swedish accounting law (Bokforingslagen), including VAT handling, tax reporting, and 7-year document retention. Multi-tenant: each user can own or be a member of multiple companies, optionally grouped into teams (for consultants).
---

# CLAUDE.md — gnubok

## Project Overview

gnubok is a Swedish-focused accounting SaaS for sole traders (enskild firma) and limited companies (aktiebolag). It implements double-entry bookkeeping compliant with Swedish accounting law (Bokforingslagen), including VAT handling, tax reporting, and 7-year document retention. Multi-tenant: each user can own or be a member of multiple companies, optionally grouped into teams (for consultants).

**Tech stack**: Next.js 16.1.5 (App Router), React 19.2.3, TypeScript 5 (strict), Zod 4, Supabase (PostgreSQL + RLS + email/password + TOTP MFA auth), Tailwind CSS 4 + shadcn/ui, Vercel hosting, Docker (self-hosted).

**Integrations**: Enable Banking (PSD2), TIC Identity (company lookup), Anthropic SDK, AWS Bedrock (`@aws-sdk/client-bedrock-runtime` for inbox smart-match), OpenAI (embeddings), Resend (email), Sentry (error tracking), Svix (webhooks), web-push (notifications), Upstash Redis + Ratelimit, Google Drive (cloud backup via OAuth), JSZip (archive export), sharp (image processing), Framer Motion (animations), Recharts (charts), PDF.js (`pdfjs-dist`), `@react-pdf/renderer` (invoice PDFs), xlsx, fuse.js (fuzzy search), ics (iCal feeds).

**Path alias**: `@/*` maps to the project root. **Language**: All code, comments, and commit messages in English. **License**: AGPL-3.0-or-later.

---

## Commands

```bash
npm run dev              # Start dev server (runs setup:extensions first)
npm run build            # Production build (runs setup:extensions first)
npm run lint             # ESLint
npm test                 # Run all Vitest tests
npx vitest run <dir>     # Run tests in a specific directory
npm run setup:extensions # Regenerate extension registry from extensions.config.json
```

---

## Key Architectural Relationships

- **Multi-tenant model**: `companies` table owns all business data. `company_members` links users to companies with roles (owner/admin/member/viewer). `teams` group companies for consultants. Company context resolved via cookie (`gnubok-company-id`) in middleware (`lib/supabase/middleware.ts`).
- **All journal entry creation** routes through `lib/bookkeeping/engine.ts`. Lifecycle: `createDraftEntry()` → `commitEntry()` (atomic voucher assignment via `commit_journal_entry` DB RPC). Convenience: `createJournalEntry()` does both. Reversal via `reverseEntry()`. Correction via `correctEntry()` in `lib/core/bookkeeping/storno-service.ts`.
- **API routes** that emit events must call `ensureInitialized()` (from `lib/init.ts`) at module level. This loads extensions, wires event handlers, and registers the supplier invoice handler + event log handler.
- **Event bus** (`lib/events/bus.ts`) is a module-level singleton. Handlers run via `Promise.allSettled` — failing handlers never crash the emitter. 36 event types defined in `lib/events/types.ts`. The event log handler persists actionable events to `event_log` table for external automation.
- **Supabase clients**: browser (`lib/supabase/client.ts`), server with cookies (`createClient()` from `server.ts`), service role (`createServiceClient()`), cookieless service role for API key auth (`createServiceClientNoCookies()` from `lib/auth/api-keys.ts`). Pagination helper: `fetchAllRows()` in `lib/supabase/fetch-all.ts`.
- **Extension system**: Opt-in via `extensions.config.json`. Core builds and runs with zero extensions. Currently enabled: `enable-banking`, `email`, `arcim-migration`, `tic`, `mcp-server`, `cloud-backup`.
- **Core reports** (in `lib/reports/`, not extensions): balance sheet, income statement, trial balance, general ledger, AR/supplier ledger, AR/supplier reconciliation, VAT declaration, journal register, monthly breakdown, continuity check, opening balances, KPI (+ definitions), NE-bilaga, INK2 declaration, SIE export, full archive export, salary journal, vacation liability, avgifter basis (employer contributions).
- **Types**: All shared types in `types/index.ts` (~2,570 lines, single source of truth). Import via `import type { T } from '@/types'`. Event types live in `lib/events/types.ts`. Extension types in `lib/extensions/types.ts`.
- **Error messages**: `lib/errors/get-error-message.ts` maps technical errors to Swedish user messages (Zod → Postgres → HTTP → context fallback).

---

## Multi-Tenant Architecture

### Data Model

- **companies**: Business unit (name, org_number, entity_type, created_by, team_id). All business data (journal entries, invoices, transactions, etc.) has a `company_id` column.
- **company_members**: Links users to companies (company_id, user_id, role, source='direct'|'team'). Roles: `owner`, `admin`, `member`, `viewer`.
- **teams**: Consultant grouping (name, created_by). A company can belong to one team. Team members auto-sync to company_members via DB triggers.
- **team_members**: Links users to teams (team_id, user_id, role='owner'|'admin'|'member').
- **user_preferences**: Stores `active_company_id` per user.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erp-mafia/gnubok](https://github.com/erp-mafia/gnubok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->

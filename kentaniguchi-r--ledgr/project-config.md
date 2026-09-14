---
trigger: always_on
description: Guidance for AI coding agents working in this repository. This is the canonical
---

# AGENTS.md

Guidance for AI coding agents working in this repository. This is the canonical
instructions file; `CLAUDE.md` imports it.

## Project Overview

**Ledgr** — a self-hostable, open-source personal finance app (AGPLv3).

Self-hosting is the only deployment model. There is no hosted product, so there
is one audience and one setup path; `docs/superpowers/specs/2026-07-07-ledgr-hosted-beta-design.md`
describes a direction that was abandoned.

Design docs live in `docs/superpowers/specs/` (design) and
`docs/superpowers/plans/` (execution). They are point-in-time records, not a
maintained spec — when a doc and the code disagree, the code wins.

## Stack

| Layer | Choice |
|-------|--------|
| Framework | Next.js 16 (App Router) |
| Language | TypeScript |
| UI | shadcn/ui v4 (`base-nova` style, Base UI primitives) + Tailwind v4 |
| Charts | Recharts v3 via shadcn Chart (`components/ui/chart.tsx`) |
| ORM | Drizzle ORM 0.45 |
| Database | PostgreSQL 18 (via node-postgres Pool) |
| Auth | Better Auth (+ passkeys) |
| Bank Sync | Plaid Node SDK and SimpleFIN — both first-class; CSV/OFX import for the rest |
| AI | Vercel AI SDK (BYOK — user brings own API key) |
| MCP | Ledgr exposes itself as an MCP server (`src/lib/mcp/`) with OAuth |
| Scheduling | `node-cron` scheduler (`src/lib/scheduler/`) driving job functions |
| Testing | Vitest + fast-check + Playwright + Stryker + MSW |

Note the UI primitives are **Base UI**, not Radix. APIs differ — `ToggleGroup`
takes `value: string[]` and hands back an empty array when the active item is
clicked again, `PopoverTrigger` takes a `render` prop, and so on. Read the
component in `src/components/ui/` before assuming a Radix signature.

## Key Conventions

- **All monetary amounts are INTEGER (cents).** $12.50 → 1250. Never use floats
  for money. Convert to display format at the UI layer via `lib/money.ts`.
- **Plaid amount convention:** Positive = debit/expense, negative = credit/income.
  `normalized_amount` flips sign for human display.
- **Ownership enforcement:** Use `scopedQuery(householdId)` to auto-inject
  `household_id` filtering. Never write manual WHERE clauses for tenant
  isolation. It takes an optional second `db` argument for tests.
- **Encryption:** Plaid/SimpleFIN tokens and AI API keys are encrypted at the app
  layer (aes-256-gcm). Keys are **versioned** — `ENCRYPTION_KEY` is v1,
  `ENCRYPTION_KEY_V2` and up are later versions, so rotation can re-wrap
  ciphertext without downtime (`pnpm rotate-keys`).
- **Timestamps:** Use `new Date()` for Postgres `timestamp` columns. Use
  `nowISO()` from `@/lib/date-utils` only for text date columns. Never
  `new Date().toISOString()` for timestamp columns — Drizzle handles the
  Date→Postgres conversion.
- **Transfers are excluded from spend.** Rows with `isTransfer` are left out of
  reports, budgets and spending totals. Investment-account activity is tagged
  `isTransfer: true` with `transferSource: "investment_account"` at sync time,
  which is what keeps brokerage fills out of the Transactions tab.
- **Deployment target:** Docker, self-hosted. `docker compose up` starts Postgres
  and the app; migrations run on container startup via
  `scripts/docker-entrypoint.sh`.

## Commands

```bash
# Development
pnpm install                     # Install dependencies
pnpm dev:db                      # Start Postgres (Docker)
pnpm dev:setup                   # Start Postgres + migrate + dev server
pnpm dev                         # Next.js dev server (requires running Postgres)
pnpm db:generate                 # Generate Drizzle migrations
pnpm db:migrate                  # Run migrations
pnpm db:studio                   # Open Drizzle Studio

# Testing
pnpm test                        # Vitest unit + integration
pnpm test:changed                # Only tests related to changed files (fast loop)
pnpm test:watch                  # Watch mode
pnpm test:coverage               # v8 coverage report
pnpm test:e2e                    # Playwright
pnpm test:mutate                 # Stryker (full)
pnpm test:mutate:incremental     # Stryker (changed files)
pnpm test:mutate:diff            # Stryker (diff vs main) — what CI runs on PRs
pnpm lint                        # ESLint
pnpm typecheck                   # tsc --noEmit

# Operations
pnpm reset-password --check|--set <email>   # Operator password check/reset
pnpm rotate-keys                            # Re-wrap encrypted columns to a new key version
pnpm backfill-clean-names                   # Backfill merchant-cleaned names
pnpm backfill-transfers                     # Backfill transfer pairing
pnpm backfill-investment-activity           # Tag existing investment rows as transfers
pnpm backfill-balances                      # Backfill balance history
pnpm build:mcp-widgets                      # Build the MCP app widgets
```

## Project Structure

```
src/
├── app/
│   ├── (auth)/                 # Login, signup
│   ├── (dashboard)/            # accounts, transactions, budgets, bills,
│   │                           # investments, reports, rules, import, settings
│   ├── api/                    # ai/chat, auth, dashboard, export, health,
│   │                           # import, mcp/oauth, plaid/{webhook,oauth-return}, search

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KenTaniguchi-R/ledgr](https://github.com/KenTaniguchi-R/ledgr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->

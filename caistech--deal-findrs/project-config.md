---
trigger: always_on
description: > This file defines project-specific conventions and workflow rules.
---

# CLAUDE.md — DealFindrs

> This file defines project-specific conventions and workflow rules.
> It EXTENDS the global guardrails at `~/.claude/CLAUDE.md` — read that first.
> Do NOT repeat global rules here. Add only what is specific to this project.

---

## Risk Tier: REVENUE

This project holds real company data, financial assessments (property deals, development
finance packs), and Stripe billing state. Apply high read:edit discipline. Shared module
changes require review of all consumers. Deployment errors must be resolved before moving
to new features.

---

## Project Purpose

DealFindrs is a Next.js 14 SaaS for property development opportunity assessment.
Core flows: opportunity creation → AI assessment → RAG status → DevFinance (QS → Valuation
→ Feasibility → Finance Pack). Secondary: voice agent data capture (ElevenLabs), Stripe
billing, company/user management.

## Stack

- **Framework:** Next.js 14 (App Router)
- **Database/Auth:** Supabase (postgres + RLS + Supabase Auth)
- **AI:** OpenAI SDK (direct, not via MCP)
- **Voice:** ElevenLabs SDK + webhook callbacks
- **Billing:** Stripe (webhooks + checkout)
- **Deployment:** Vercel

---

## AUTHENTICATION CONTRACT (non-negotiable)

Every API route handler MUST call `requireAuth()` before any data operation.
The only exceptions are: `stripe/webhook` (uses Stripe signature verification instead)
and ElevenLabs webhooks (use shared secret header validation).

```typescript
// Required pattern for all API routes
import { requireAuth } from '@/lib/auth/require-auth'

export async function POST(request: NextRequest) {
  const { user, error } = await requireAuth(request)
  if (error) return NextResponse.json({ error }, { status: 401 })
  // ... rest of handler
}
```

Never use `request.headers.get('x-user-id')` as an auth mechanism.
Never fall back to a hardcoded user ID string.

---

## SUPABASE PATTERNS

### Client initialization
Always use the lazy-init factory pattern:
```typescript
let _client: SupabaseClient | null = null
function getSupabaseAdmin(): SupabaseClient {
  if (!_client) {
    const url = process.env.NEXT_PUBLIC_SUPABASE_URL
    const key = process.env.SUPABASE_SERVICE_ROLE_KEY
    if (!url || !key) throw new Error('Supabase credentials not configured')
    _client = createClient(url, key)
  }
  return _client
}
```
Never initialize at module level with `!` assertions (causes silent startup failures
in edge environments where env vars may not be available at import time).

### Service role key
Used server-side only. Never import or reference in client components.
Used legitimately in: all `/api/*` route handlers, `src/lib/devfinance/db.ts`.
If you are adding a new client component that needs data, use a server action or
route handler — never pass the service role key to the client.

### RLS
All Supabase tables MUST have RLS enabled. The devfinance schema uses company-scoped
policies via `get_user_company_id()`. Follow this pattern for any new tables.
Check the devfinance-schema.sql for the canonical policy structure.

---

## API ROUTE CONVENTIONS

```
src/app/api/
  opportunities/         — CRUD for property opportunities
  devfinance/            — QS, valuation, feasibility, affordable gap, finance pack
  assess/                — AI assessment engine
  voice/                 — ElevenLabs voice chat
  webhooks/elevenlabs/   — Voice agent data callbacks
  stripe/                — Checkout, portal, webhook
  company/               — Company creation, invite, join
  admin/                 — User management (requires admin role check)
  site-intel/            — Address-based property intelligence
  abn-lookup/            — ABN validation
```

Route files are named `route.ts`, not `index.ts`.
Export named `GET`, `POST`, `PUT`, `DELETE` — no default export.
Always type the return value: `NextResponse.json<T>(...)`.

---

## DEVFINANCE MODULE ORDER

Modules are generated in strict dependency order:
1. QS Report (construction cost estimate)
2. Valuation Report (GRV, PRSV — requires QS for TDC)
3. Feasibility Study (cash flow, sensitivity — requires QS + Valuation)
4. Affordable Gap Analysis (optional — requires Feasibility)
5. Finance Pack (export — requires all above)

Never skip steps or generate a downstream module without the upstream module being present.
Use `getQSReport()` / `getValuationReport()` from `src/lib/devfinance/db.ts` to resolve
prior versions.

---

## ERROR HANDLING CONVENTIONS

### Webhook helpers
Helper functions called from webhook handlers MUST return `{ error }` or throw — never
silently swallow database operation failures. The top-level handler should surface these:
```typescript
const { error } = await supabase.from('companies').update(...).eq(...)
if (error) {
  console.error('[webhook] company update failed:', error.message)
  // decide: throw to return 500, or continue with degraded state (document why)
}
```

### Side-effect failures
If a secondary operation fails (e.g. activity log insert, profile update), document
explicitly whether this is intentionally non-fatal and why. Do not use
`// Don't fail - X was created` without a structured log.

### Async in webhooks
All webhook helper functions must be `async` and `await`ed. Never fire-and-forget
database writes inside webhook handlers.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caistech/deal-findrs](https://github.com/caistech/deal-findrs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->

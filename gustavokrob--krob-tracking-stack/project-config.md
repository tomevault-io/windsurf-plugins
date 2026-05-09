---
trigger: always_on
description: > This file is Claude Code's anchor. It is loaded into the context of every
---

# Tracking Stack Template

> This file is Claude Code's anchor. It is loaded into the context of every
> conversation in this repo. Keep it under ~200 lines. Put detailed reference
> in `docs/` and procedural walkthroughs in `.claude/skills/`.

## What this repo is

A Cloudflare Pages + D1 tracking stack that captures first-party attribution
data, fires server-side conversion events to Meta CAPI / GA4 / Google Ads,
and ships a built-in dashboard that shows revenue, products, UTMs, and
tracking health. It replaces Stape + GTM Server-Side for creators running
paid traffic to lead or sales pages.

Each recipient deploys their own copy in their own Cloudflare account with
their own D1 database. There is no shared backend. The template is designed
to be unpacked into a blank folder and driven from Claude Code.

**The stack does three things:**

1. **Server-side conversions** with ITP-resistant identifier capture (400-day
   first-party cookies, edge-set `fbp`/`fbc`, GA4 client ID parsing, event
   deduplication between browser pixel and server CAPI).
2. **First-party attribution persistence** — every lead and every purchase
   stores its UTMs, `fbp`/`fbc`, `gclid`, and originating session, so the
   dashboard can show where each conversion came from.
3. **A self-contained dashboard** at `/dash` with six sections: revenue,
   product sales, paid-traffic attribution with Meta spend/CPA/ROAS, UTM
   breakdown, recent leads with UTMs, and tracking health (ITP recovery,
   adblock recovery, bot filters).

## Triage: what to do when a recipient starts a conversation

If the recipient has **just unpacked the template into a blank folder** and
the repo is otherwise untouched, invoke the `deploy-stack` skill.

If a recipient is asking **"is my tracking working"** or **"why isn't my
Meta dashboard showing conversions"**, invoke `verify-tracking`.

If they want to **add another lead page or sales page**, invoke `add-page`.

If they say **"I use [sales platform not in Eduzz/Hotmart/Kiwify]"**, invoke
`add-sales-platform`.

For anything else, ask a clarifying question before reaching for a skill.

## Identifier chain

Every visit generates identifiers at the edge, persists them to D1, and
threads them through the checkout flow so the webhook can enrich the
purchase with its original attribution.

| Identifier | Origin | Storage | Used by |
|---|---|---|---|
| `_krob_sid` | Middleware, UUID per visit | 400d cookie + `sessions` row | Joins every event to its originating visit |
| `fbp` | Middleware, Meta spec `fb.2.{ts}.{rand}` | Cookie + `sessions.fbp` | Meta CAPI |
| `fbc` | Middleware, derived from `fbclid` URL param | Cookie + `sessions.fbc` | Meta CAPI |
| `ga_client_id` | Parsed from GA4's `_ga` cookie at edge | `checkout_sessions.ga_client_id` | GA4 Measurement Protocol |
| `trk` | Sales page JS, UUID per checkout intent | `checkout_sessions.trk` (unique) | Webhook lookup after purchase |
| `event_id` | Client, UUID per event | `event_log`, `purchase_log` | Dedup between browser pixel and server CAPI |
| `external_id` | Middleware, UUID per visitor | Cookie + `sessions.external_id` | Meta Advanced Matching |

**The `trk` chain is the critical one for sales pages**: generated on the
sales page visit → persisted to `checkout_sessions` with all attribution →
passed to the sales platform as a custom field (`tracker.code1` for Eduzz,
`xcod` for Hotmart, `sck` for Kiwify) → returned in the webhook payload →
looked up to enrich the Meta/GA4/Google Ads conversion.

Hop-by-hop debugging bible: `docs/data-flow.md`

## Hard rules (do not violate)

- **Never log PageView events to `event_log`.** PageView still fires to
  Meta/GA4 — it just doesn't write to D1. This keeps per-instance D1 write
  volume sustainable forever. Enforced at `functions/tracker.js:118`.
- **Never commit secrets.** `wrangler.toml`, `.dev.vars`, `.env*` are all
  gitignored. Only `wrangler.toml.example` is tracked. Product configuration
  (`config/products.js`) IS tracked — product IDs and tag IDs are not secrets.
- **Always use parameterized SQL.** Every D1 query uses `.bind()`. No string
  interpolation of user input, ever.
- **Hash PII before sending to ad platforms.** Email, phone, name get SHA-256
  hashed after lowercase-and-trim normalization (phone additionally strips
  non-digits and leading zeros). Raw PII persists in D1 for debugging only
  and never leaves the recipient's own infrastructure.
- **Per-platform webhook adapter pattern.** Each sales platform has its own
  file at `functions/webhook/<platform>/[slug].js` that gates on the URL
  slug, then parses the platform's payload into the normalized shape. The
  shared lookup/enrichment/fan-out lives in `functions/webhook/_core.js`.
  When adding a new platform, copy an existing adapter as a structural
  reference — do not add platform branching to `_core.js`.
- **Webhook endpoints are gated by an obscure URL.** Every platform has a
  per-recipient UUID v4 slug stored as `env.<PLATFORM>_WEBHOOK_SLUG`. The
  adapter's full path is `/webhook/<platform>/<slug>`; wrong slug returns
  404 (indistinguishable from a missing route). `deploy-stack` generates
  the UUIDs automatically and prints the full URLs for the recipient to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gustavokrob/krob-tracking-stack](https://github.com/gustavokrob/krob-tracking-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

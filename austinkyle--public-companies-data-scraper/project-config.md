---
trigger: always_on
description: > Root context file. Re-injected every turn. Keep it lean — every token here is a recurring tax.
---

# CLAUDE.md — AI PE Deal Screener (Capstone Build)

> Root context file. Re-injected every turn. Keep it lean — every token here is a recurring tax.
> Single source of truth for system-wide invariants, architecture map, UX spec, and build sequence.
> Module detail lives in each subdirectory's CLAUDE.md. Contracts live in `/specs` and are FROZEN.
> Hard code lives in `/reference` — COPY it, do not reinvent it (see §6).

---

## READ ORDER (token-efficiency rule — enforced every session)

1. Read THIS file first, every session. It is the Map.
2. Then read ONLY the `CLAUDE.md` of the ONE module relevant to the current task (`/backend`, `/mcp-server`, or `/frontend`). Each module CLAUDE.md contains a **routing table** telling you exactly which files to READ, which to SKIP, and which tools to load per task type. Obey it.
3. NEVER load all workspaces, the full `/specs` set, or another module's internals. The HTTP/MCP contracts in `/specs` are the only cross-module interface — if you think you need another module's source, you're wrong; read its contract instead.

---

## 0. What this is (scope — do not exceed)

An **autonomous AI Private Equity Deal Screener**, built as a **portfolio capstone** (with possible small-scale use by known users — not a SaaS focus).

Full agentic system: niche keyword → candidate discovery (SEC EDGAR) → MCP-served financial reading → LLM screening against criteria → ranked report → Stripe-gated full results → Telegram alerts.

**Product model (BuiltWith pattern):** one free lookup with partial results, full results + monitoring behind Stripe. Demo data source is **SEC EDGAR only** (public, free, legal). Private-source connectors are documented as future extensions, NOT built.

**Out of scope for v1:** private deal-database integrations, multi-tenant orgs, reselling scraped proprietary data, trading functionality.

---

## 1. Non-negotiable invariants

- **Human-in-the-loop.** Output is a first-pass research filter, never a buy/sell decision.
- **Never assert unverified financials.** Every figure carries `verified: false` + `source_url`; UI shows the source link beside every number.
- **Contracts in `/specs` are frozen.** No changes without explicit instruction + a dated entry in `DECISIONS.md`.
- **No secrets in code or git.** Env vars only; ship `.env.example` with placeholders.
- **No raw HTML/JSON dumps to the LLM.** Deterministic parsing first; the model gets only distilled structured data.
- **Stripe = hosted Checkout + Customer Portal only.** No custom card forms. Entitlement driven by verified webhooks only.
- **Freemium gating is SERVER-SIDE.** The API truncates/blurs data for free users before it leaves the backend. Never send full data to the client and hide it with CSS.
- **SEC compliance:** every EDGAR request sends the declared `User-Agent` with contact email; global rate limit ≤ 8 req/s. (Implemented in `/reference/edgar_client.py` — use it.)

---

## 2. Architecture map

```
[Next.js frontend] --HTTP--> [FastAPI orchestrator] --calls--> [Screening Agent (LLM)]
                                      |                                |
                                      |--> [Source layer: EDGAR client] (discovery: full-text search + SIC)
                                      |--> [MCP server] (companyfacts XBRL -> FinancialSnapshot)
                                      |--> [Stripe] (Checkout + webhook entitlement)
                                      |--> [Telegram alerter]
                                      |--> [Postgres] (users, screens, candidates, results, entitlements, stripe_events)
                                      |--> [Async jobs] (screens write results row-by-row as they finish)
```

Components communicate ONLY through `/specs` contracts. No component imports another's internals.

---

## 3. WAT framework (full version in `/docs/WAT.md`)

**Workflows:**
- `run_screen`: keyword/criteria → EDGAR discovery → financials via MCP → agent screens each candidate (concurrent, bounded) → results persist **one row at a time as they complete** → rank → Telegram alert on completion.
- `recurring_monitor`: scheduled re-run of saved screen → diff vs prior → alert only on new matches.
- `billing`: Checkout → verified webhook → entitlement set/cleared → gates depth of results.
- `onboarding`: free lookup, see partial report, connect Telegram + pay to unlock.

**Agent:**
- Evaluates ONE candidate against the user's criteria; returns structured `ScreenResult` via **forced tool use** (see `/reference/agent_structured_output.py`).
- Inputs: structured candidate + `FinancialSnapshot` + criteria. Never raw filings.
- Guardrails: never invent figures; missing data → flag, don't guess; output must validate against schema or it is retried, never stored.

**Tools:** EDGAR discovery client, MCP financial reader, Stripe handlers, `telegram_send`, typed DB repositories.

---

## 4. Frozen contracts (full schemas in `/specs`)

**Models (`/specs/SCHEMA.md`):**
- `User`: id, email, telegram_chat_id, stripe_customer_id, entitlement_status
- `ScreenRequest`: id, user_id, keywords, sic_codes[], revenue_min/max, deal_size, custom_flags[], schedule(none|daily), status(queued|running|complete|failed)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [austinkyle/public-companies-data-scraper](https://github.com/austinkyle/public-companies-data-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->

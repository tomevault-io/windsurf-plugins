---
trigger: always_on
description: You are part of Lavern v0.15.0, a multi-agent legal design system that transforms
---

# Lavern — A multi-agent legal system. Yours.

## System Identity

You are part of Lavern v0.15.0, a multi-agent legal design system that transforms
legal documents through collaborative AI analysis and human-centered design.
Lavern is an open-source multi-agent legal system. The "law firm" framing used
in some docs is an architectural analogy, not a description of what Lavern is.
Lavern is not a law firm and does not provide legal advice.

The codebase is called "The Shem" (the name inscribed in the golem's mouth).
The product is called "Lavern". These names are interchangeable in internal docs.

## Shared Principles

1. Legal effect must remain identical after transformation
2. Every finding must cite specific text as evidence
3. Debate is a feature, not a bug — agents should challenge each other
4. Human gates are mandatory, never skip them
5. Every engagement ships with an audit bundle alongside the deliverable: structured findings, debate resolutions, verification results, and cost log. (The synthesis-editor agent is architected to produce a separately polished "review package" output; in practice that lives across the audit bundle files rather than as a single rendered document.)

## Non-Negotiable Preservation Categories

- Monetary amounts, liability caps, penalties
- Time periods, notice requirements, deadlines, cure periods
- Jurisdiction, governing law, venue, arbitration
- Dispute resolution mechanisms, termination triggers
- Defined terms with specific legal scope
- Insurance coverage requirements
- Regulatory compliance language

## Disclaimer

This system assists with document design and accessibility.
It does not provide legal advice. Always verify redesigned documents
with qualified legal professionals.

## Project Structure

### Core Engine
- `src/agents/` — 67 agent prompts (59 specialists + 7 orchestrators + 1 base), 59 agent definitions
- `src/agents/profiles.ts` — 63-agent profile registry (skill ratings, personality, DiceBear avatars)
- `src/mcp/tools/` — 21 MCP tool modules (debate board, scoring, verification, memory, risk pricing, baselines, knowledge base, report cards, quality checks, handoffs, feedback loop, document reader)
- `src/mcp/remote-bridge/` — JSON-RPC 2.0 HTTP bridge exposing 12 Counsel tools for Anthropic Managed Agents; shared-secret auth, per-session dispatch, Zod arg validation (gated by `LAVERN_MANAGED_AGENTS_BRIDGE=1`)
- `src/hooks/` — Audit logging, human gate enforcement, cost tracking
- `src/router/` — LLM-based request router with deterministic fallback and template mapping
- `src/orchestrator.ts` — Core orchestration loop (dispatch agents, manage turns)
- `src/dispatch.ts` — Session dispatch (workflow selection, gate resolver, budget)
- `src/permissions/` — Phase-based dynamic tool permissions
- `src/session/` — Session state management + session manager (lifecycle, TTL, eviction)
- `src/events/` — Event bus for real-time streaming
- `src/gates/` — Human gate resolvers (readline CLI, async API, webhook, auto-approve)
- `src/config.ts` — Centralized configuration (all settings env-var backed)
- `src/utils/` — Shared utilities (atomic fs writes, message streaming, error recovery)
- `src/types/` — TypeScript type definitions and Zod schemas
- `SOUL.md` — Default firm personality (CLI/Claw fallback; browser users set soul in My Page)

### Workflows
- `src/workflows/` — 9 workflow templates:
  - `counsel` — Quick legal questions
  - `review` — Full contract review with debate
  - `adversarial` — Builder + attacker + synthesizer
  - `roundtable` — Parallel expert panel + debate + synthesis
  - `legal-design` — Legal design transformation
  - `full-bench` — Maximum team engagement
  - `pre-engagement` — Intake and team selection
  - `verification` — Standalone document verification pipeline
  - `tabulate` — Tabular multi-document review (one row per doc, every cell cited)
- `src/workflows/executor.ts` — Generic workflow runner with soul + personality injection

### API Server
- `src/api/` — Fastify API server with WebSocket event streaming
  - `src/api/middleware/` — Auth (LOCAL-MODE no-op; cookie/Bearer logic preserved for `LAVERN_AUTH_ENABLED=true`), Zod validation, x402 payment
  - `src/api/routes/` — 27 route modules. The auth-shaped ones (auth-routes, google-auth, billing, referral) only register when `LAVERN_AUTH_ENABLED=true`; in default LOCAL MODE the dashboard runs as the synthetic `local-user` and those routes 404.
    - `sessions.ts` — Session CRUD + gate decisions + soul injection from user profile
    - `engage.ts` — Agent-native engagement (sync + webhook modes)
    - `verify.ts` — Standalone document verification
    - `matters.ts` — Matter management (engagements, team selection)
    - `briefing.ts` — LLM-powered briefing analysis for intake
    - `auth-routes.ts` — User signup, login, logout, profile (gated)
    - `google-auth.ts` — Google OAuth login/signup (gated)
    - `billing.ts` — Stripe-backed billable-hours subscription (gated)
    - `referral.ts` — Referral stats (gated)
    - `claw.ts` — Clawern remote monitoring & control
    - `challenge.ts` — Lavern Challenge blind document comparison
    - `challenge-prompt.ts` — Challenge prompt builder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnttiHero/lavern](https://github.com/AnttiHero/lavern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

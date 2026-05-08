---
trigger: always_on
description: **Repo:** 8gi-foundation/8gent-code
---

# 8gent Code - Agent Context

**Repo:** 8gi-foundation/8gent-code
**Domain:** 8gent.dev
**Runtime:** Bun (not Node)
**Stack:** Bun, Ink v6 (React for CLI), SQLite + FTS5, TypeScript

## Repo-Specific Rules

- Bun everywhere. Never use Node or npm in scripts.
- Run `bun run tui` to test before any push. Never push untested.
- Run `bun run benchmark:v2` for capability regression.
- TUI uses Ink v6 - React component model but for terminal.
- Provider model is ADAPTIVE ROUTER, not a single provider. Do not describe as "Ollama default".
- 11 providers wired. See `packages/providers/` for registry.
- NemoClaw policy engine is deny-by-default (`packages/permissions/policy-engine.ts`).
- Never add `Co-Authored-By` to commits. Never reference AI vendors in code or commits.

## Key Files

| File | Purpose |
|------|---------|
| `packages/eight/tools.ts` | Core tool definitions |
| `packages/eight/agent.ts` | Agent loop, abort, checkpoint restore |
| `packages/eight/prompts/system-prompt.ts` | System prompt |
| `packages/permissions/policy-engine.ts` | NemoClaw deny-by-default |
| `packages/memory/store.ts` | SQLite + FTS5 memory |
| `packages/providers/failover.ts` | Failover chain |
| `apps/tui/` | Terminal UI entry point |

## Provider Failover Chain

local 8gent (localhost) -> local Qwen (localhost) -> OpenRouter `:free`

`auto:free` = dynamically picks best OpenRouter `:free` model.
Apple Foundation auto-enables on macOS 26 + Apple Silicon + bridge binary.

---

# 8GI Ecosystem Context

> Canonical source of truth. Auto-propagated to all 8GI repos on every push to `8gi-governance` main.
> Last updated by: CI auto-sync. Edit here, not in individual repos.

---

## Mission

Democratize infinite general intelligence for everyone. Free, local-first, privacy-preserving.

## The 8 Principles

1. Design first, not last. Friction is the enemy.
2. Free and local by default. No API keys to start. Cloud is opt-in.
3. Self-evolving. Skills accumulate. Lessons persist.
4. Hyper-personal. Learn the user's patterns, preferences, codebase, style.
5. Accessible. Voice input, screen readers, audio docs. Adapt to the user.
6. Orchestrate by default. Delegate to sub-agents. Decompose complexity.
7. Reduce friction, increase truth. Prefer voice and conversation over forms.
8. The work speaks for itself. Ship, don't sell. Evidence, not enthusiasm.

---

## Product Ecosystem

| Product | Domain | Repo | Stack | Status |
|---------|--------|------|-------|--------|
| **8gent OS** | 8gentos.com / {user}.8gentos.com | 8gent-OS | Next.js 16, Convex, Clerk, Stripe | Active - Wave 4 |
| **8gent Code** | 8gent.dev | 8gent-code | Bun, Ink v6 (React TUI) | Shipped v0.13.0 |
| **8gent Jr** | 8gentjr.com | 8gentjr | Next.js, Convex, Clerk | Active - COPPA compliant |
| **8gent World** | 8gent.world | 8gent-world | Astro/Next.js | Docs + ecosystem story |
| **8GI Foundation** | 8gi.org | 8gi-governance | Next.js, Convex, Clerk | Auth-gated inner circle |
| **8gent App** | 8gent.app | 8gent | TBD | Concept stage |

---

## Architecture Decisions (binding across all repos)

### Infrastructure
- **Primary cloud**: Vercel (Next.js hosting) + Hetzner cax21 (self-hosted daemon/vessels)
- **Database**: Convex (multi-tenant via `tenantId` field on every table)
- **Auth**: Clerk (prod + dev)
- **Billing**: Stripe
- **Storage**: Hetzner Object Storage
- **Vessel runtime**: Fly.io Amsterdam (eight-vessel.fly.dev) - parallel to Hetzner, not replaced yet

### 8gent OS Tenant Model
- Each user gets `{username}.8gentos.com`
- Wildcard Vercel domain routes to Next.js `[username]` dynamic route
- Convex row-level multi-tenancy via `tenantId`
- Per-user: mini-apps, marketplace installs, skills, memory, voice

### Provider Stack (8gent Code)
- Adaptive router, NOT a single provider
- Default active: `8gent` (localhost) + `ollama`
- 11 providers wired: `8gent`, `ollama`, `openrouter`, `groq`, `grok`, `openai`, `anthropic`, `mistral`, `together`, `fireworks`, `replicate`
- Failover: local 8gent -> local Qwen -> OpenRouter `:free`
- Apple Foundation: auto-enables on macOS 26 + Apple Silicon

---

## Hard Rules (NON-NEGOTIABLE - all repos, all agents)

### Code
- No em dashes anywhere. Use hyphens or rewrite.
- No purple/pink/violet (hues 270-350) in any UI.
- No secrets in chat or commits. File-based injection only.
- No AI vendor names in any surface (no Claude, Anthropic, OpenAI, Hermes, Nous).
- No Co-Authored-By in git commits. James owns 100% of all work.
- No direct push to main. Always feature branch + PR.
- Post-push Vercel check mandatory: HTTP 200 + screenshot + Telegram. This is the definition of done.
- Bun, not Node, for all 8GI runtimes.
- Test before pushing. Never push untested code.

### Content / Copy
- No customer-facing AI/tooling language. No "Export for AI", "Send to Claude", etc.
- No em dashes in any publication.
- No invented biography or statistics about James.
- No self-harm details about Nicholas in any public content.
- No formal diagnosis claims (James is self-identified AuDHD, not formally diagnosed).
- aidhd.dev is stealth mode - do not mention publicly.

### Process
- Every change gets a GitHub issue first. Link PR to issue with `Closes #N`.
- All work tracked at: https://github.com/orgs/8gi-foundation/projects/1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [8gi-foundation/8gent-code](https://github.com/8gi-foundation/8gent-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

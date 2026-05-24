---
trigger: always_on
description: This workspace contains three completely separate products. They share engines and a contract, but they must never be mixed.
---

# VCSM Workspace — Global Rules

This workspace contains three completely separate products. They share engines and a contract, but they must never be mixed.

## The Three Apps

| App | Path | Product |
|-----|------|---------|
| **VCSM** | `apps/VCSM/` | Social marketplace hybrid platform (Instagram + Airbnb) |
| **Wentrex** | `apps/wentrex/` | Standalone multi-tenant LMS SaaS |
| **Traffic** | `apps/Traffic/` | Programmatic SEO directory engine (Next.js) |

**VCSM** is the core platform — a social commerce app where creators and service providers manage actor-based identities (personal profiles or business VPORTs), post content, chat, book services, and run storefronts. External business sites like Tripoint Lock & Keys (tripointlockandkeys.com) consume their VPORT data from VCSM via Edge Function APIs, keeping their own domain and UI while VCSM remains the source of truth for business identity, services, reviews, and booking.

**Traffic** is a standalone Next.js 14 static site that generates indexable city/service/neighborhood/provider directory pages for organic search discovery, routing visitors back to the VCSM platform via deep links with tracking parameters. It currently runs entirely on mock data with no database, authentication, or engine imports — it is self-contained and deployment-ready at `traffic.vibezcitizens.com` once data is wired.

## Non-Negotiable Rules

- **Never import from one app into the other.** `apps/VCSM` and `apps/wentrex` are fully isolated products.
- **Never assume a pattern from one app applies to the other.** They share contracts and engines, but have different domain models, UI structures, and product logic.
- **Always confirm which app you are working in before making changes.** If the task is ambiguous, ask.
- **Never move features between apps.** If both apps need something similar, it belongs in `engines/` or `shared/`, not copied between apps.
- **Both apps have LMS features — they are not the same LMS.** VCSM has an embedded `/learning` route. Wentrex IS a standalone LMS SaaS. Do not conflate them.

## VCSM Architecture Contract — Mandatory Pre-Work Gate

> **Before working on anything inside `apps/VCSM/`, read this file in full:**
> `/Users/vcsm/Desktop/VCSM/zNOTFORPRODUCTION/_CANONICAL/zcontract/ARCHITECTURE.md`
>
> This contract is locked. It overrides any local assumptions, prior patterns, or inferred conventions.

### Identity — Actor-Based Only

- VCSM is actor-based. The canonical identity fields are `actorId` and `kind` (`'user'` | `'vport'`).
- **Never** scope behavior by `profileId`, `vportId`, or raw `userId`.
- **Never** expose `profileId` or `vportId` through `useIdentity()` or any public hook/controller surface.
- "Owner" always means Actor Owner — verified through `actor_owners`. There is no other ownership model.

### Screen Role Boundaries

Every file in `apps/VCSM/src/features/` must belong to exactly one layer and respect its role:

| Layer | Role | What it must NOT do |
|---|---|---|
| **Final Screen** | Route entry + identity gate only | No hooks, no computation, no data fetching |
| **View Screen** | Hooks + component composition | No business logic, no DB access |
| **Components** | Presentational only | No hooks, no data fetching, no side effects |
| **Hooks** | Lifecycle / timing / state wiring | No business rules, no direct DB access |
| **Controllers** | Business rules, ownership, permissions | No React, no UI concerns |
| **Models** | Domain shape translation, pure transforms | No side effects, no DB access |
| **DAL** | Raw Supabase access only | No business logic, no UI concerns |

### Mandatory Build Order

Always build in this order. Do not skip layers or work backwards.

```
DAL → Model → Controller → Hook → Components → View Screen → Final Screen
```

### Additional Hard Rules

- **Imports:** All new cross-folder imports must use `@/...` path aliases — never relative `../../` chains.
- **DAL selects:** Always use explicit column lists. `select('*')` is banned.
- **File length:** Keep files under 300 lines. If a file exceeds this, split it before continuing.
- **Cross-feature access:** One feature must never import directly from another feature's internals. All cross-feature access must go through adapters only.

---

## Shared Infrastructure (Safe to Consume from Both Apps)

- `engines/` — reusable domain engines (chat, identity, hydration, portfolio, reviews, booking, notifications)
- `shared/` — domain-neutral primitives (UI, utils, types)
- `contract/` — locked architecture contracts

## Dependency Direction

```
apps/VCSM     ──┐
                ├──→ engines/ ──→ shared/
apps/wentrex  ──┘
```

Apps never depend on each other. Ever.

---

## Technology Stack (VCSM)

- **Language:** JavaScript (ES Modules) — **TypeScript is BANNED**
- **UI:** React 19 + Vite
- **Styling:** UnoCSS + CSS custom properties (`--vc-*` tokens in `citizens-theme.css`)
- **State:** Zustand
- **Routing:** React Router DOM
- **Database:** Supabase (PostgreSQL + Auth + Realtime)
- **Config:** `jsconfig.json` — never `tsconfig.json`
- **Full stack rules:** `logan/platform/vcsm.platform.stack-rules.md`

### Forbidden in VCSM
- `.ts` / `.tsx` files — zero allowed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VibezCitizens/VCSM](https://github.com/VibezCitizens/VCSM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->

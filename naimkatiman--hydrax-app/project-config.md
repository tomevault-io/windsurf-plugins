---
trigger: always_on
description: - **Name:** hydrax-app
---

# CLAUDE.md — HydraX Institutional Workflow Platform

## Project Identity

- **Name:** hydrax-app
- **Authoritative spec:** [docs/prd.md](docs/prd.md) — read §1–§24 before scoping any module. Everything below is a compressed operating layer on top of it.
- **Positioning (PRD §1, §6):** White-label institutional workflow platform above HydraX's regulated tokenisation, trading, and custody rails. Canton-aligned, privacy-preserving, multi-party.
- **NOT building:** competing exchange, custody system, tokenisation protocol, retail trading app, DeFi frontend.
- **First wedge (PRD §23):** institutional onboarding + issuance + subscription servicing workspace for tokenized products.

## Current Repo State

- [index.html](index.html), [app.js](app.js), [styles.css](styles.css) — static HTML/JS prototype of the operator console. Reference for UX patterns (orders drill-down, venues panel, workspace persistence). **Not production code.**
- [STATE.yaml](STATE.yaml) — current working slice, verification log. Update on every progress step.
- [docs/](docs/) — PRD, plan, problems, workflow, homework, ideas.
- No backend, no database, no build pipeline yet.

## Prototype — How to Work on It Today

Until `services/` and `web/apps/` exist, the three-file prototype is the only runnable surface. Treat it as the verification target.

**Preview:**
```bash
python3 -m http.server 8000   # then open http://localhost:8000
# or just open index.html directly in a browser
```

**Verification (run after every edit — these are the smallest correctness proofs):**
- `node --check app.js` — syntax check, must pass.
- `getElementById` ↔ HTML `id=` audit — every `document.getElementById("x")` in [app.js](app.js) must have a matching `id="x"` in [index.html](index.html). Zero misses.
- CSS class audit — every class referenced in [index.html](index.html) or added via [app.js](app.js) must be declared in [styles.css](styles.css).
- `wc -l index.html app.js styles.css` — record counts in [STATE.yaml](STATE.yaml) `verification_log`.
- `git diff --stat` — confirm only the expected files changed (prototype work touches exactly those three + STATE.yaml).

**Gotchas:**
- LocalStorage keys are versioned: `hydrax.workspace.v1` ([app.js:248](app.js#L248)) and `hydrax.activity.v1` ([app.js:490](app.js#L490)). If you change the persisted shape, **bump the `.v1` suffix** — do not silently break users with stale state.
- All three prototype files change together for any interactive slice (HTML ids + JS handlers + CSS classes). A commit that touches one or two without the other is almost always incomplete.
- Go/TS verification gates below apply once those services exist. For prototype work, the five checks above are the gate.

**STATE.yaml `verification_log` entry format** (match what's already there):
`YYYY-MM-DD — <slice>: node --check app.js passes; <id audit result>; <css audit result>; wc -l index.html=N app.js=N styles.css=N; git diff --stat confirms N code files changed`

## Target Tech Stack

### Backend (polyglot microservices)

- **Go:** performance-critical core — workflow orchestration, approval chains, audit, HydraX rails adapters, Canton/Daml command + event bridge.
- **Node.js + TypeScript:** notification service, integration adapters (KYC/KYB, SSO, email, CRM), BFF for React portals.

### Frontend

- **React + TypeScript + Redux Toolkit** (RTK + RTK Query for server state).
- **Vite** for build.
- Separate role-aware shells: `issuer-portal`, `distributor-portal`, `investor-portal`, `ops-console`, `admin`.
- White-label theming via CSS variables + tenant config injected at runtime.
- **Icons: lucide-react only. No emoji in UI.**

### Data

- **PostgreSQL:** tenants, users, roles, workflow definitions, approval state, audit log, relational lookups, reporting read models.
- **MongoDB:** flexible tenant-configurable payloads, workflow state projections, document metadata, notification envelopes.

### Deploy

- **Railway:** one service per Go/Node binary, React apps as static sites, Postgres + Mongo as addons. Env per stage (dev/staging/prod).

## Target Repo Layout

```
hydrax-app/
  services/
    workflow-svc/         # Go — orchestration, state machines, SLA tracking
    approval-svc/         # Go — approval chains, escalations
    audit-svc/            # Go — action log, evidence trail
    hydrax-adapter/       # Go — HydraX rails integration
    canton-adapter/       # Go — Daml command/event bridge
    notify-svc/           # Node/TS — email, in-app, webhook
    integration-svc/      # Node/TS — KYC, SSO, CRM
    bff/                  # Node/TS — aggregates services for React
  web/
    apps/
      issuer-portal/
      distributor-portal/
      investor-portal/
      ops-console/
      admin/
    packages/
      ui/                 # shared components, lucide icons
      tenant-theme/       # white-label theming primitives
      api-client/         # RTK Query generated from BFF OpenAPI
  db/
    postgres/migrations/
    mongo/schemas/
  docs/
    prd.md                # authoritative
    plans/YYYY-MM-DD-*.md
    env.md                # every env var documented
```

## Architecture Principles (PRD §10, §13)

- Not a generic blockchain app. Privacy-preserving multi-party workflow platform.
- **HydraX = rails.** This app = workflow + experience + orchestration layer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [naimkatiman/hydrax-app](https://github.com/naimkatiman/hydrax-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

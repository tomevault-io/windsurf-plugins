---
trigger: always_on
description: Read [`/docs/README.md`](./docs/README.md) first for the full spec
---

# DamDam — Agent Instructions

Read [`/docs/README.md`](./docs/README.md) first for the full spec
index and recommended reading order. All product, technical,
security, and infrastructure decisions live in `/docs` — treat it
as the source of truth, not this file.

## Before implementing any feature

1. Find the User Story (`US-XX`) in [`/docs/prd.md`](./docs/prd.md)
   §4 and its acceptance criteria
2. Check [`/docs/testing-qa.md`](./docs/testing-qa.md) for the
   required test cases mapped to that story — write these before
   or alongside the implementation for the strict-TDD categories in
   §14.1 (check-in, SOS, payment/idempotency, auth)
3. Cross-check the relevant sections in `/docs/data-model.md`,
   `/docs/api-spec.md`, and the frontend spec for your platform
   (`/docs/frontend-mobile.md` or `/docs/frontend-dashboard.md`)

## Who implements what

This project splits work by **concern, not by whole codebase area**
— the split exists to put the small amount of genuinely taste-
sensitive work where design judgment matters, and everything else
where it's cheapest to build, without defaulting to "Claude builds
all of mobile" (that would be far more Claude usage than necessary
and works against the cost-optimization policy below).

**The core mechanism: Claude produces a design system once, Codex
implements against it, Claude reviews the rendered result.**
Codex's frontend gap is a taste/judgment gap, not a capability gap
— it's very capable of assembling already-decided design tokens,
component patterns, and layout rules precisely. Removing ambiguous
taste calls from Codex's job (by giving it an explicit system to
follow) closes most of the quality gap without Claude touching
every screen.

| Area | Primary implementer | Why |
|---|---|---|
| `docs/design-system.md` (tokens, typography, spacing, component patterns) | **Claude**, produced once, revised rarely | The one-time investment that makes everything below cheaper to get right |
| `apps/mobile/src/screens/SosConfirm/**`, `SosSent/**`, and the onboarding flow (`Splash` through `DepartureDate`) | **Claude, direct implementation** | The small, named exception — these are the screens where the *feel* of the interaction (reassurance during a real emergency, trust during first use by a low-tech-confidence demographic) matters more than typical UI, and a taste gap here has outsized real-world cost |
| All other `apps/mobile/src/screens/**`, `apps/mobile/src/components/**` | **Codex**, against `design-system.md` | Implemented by assembling the design system's already-decided pieces — Claude reviews the rendered output (see the GitHub Action below), doesn't build it |
| `apps/mobile/src/hooks/**`, `apps/mobile/src/services/**`, `apps/mobile/src/store/**` | **Codex** | Logic/state layer — no visual output |
| `apps/dashboard/**` | **Codex** | Explicitly speced as "functional, not polished" in `frontend-dashboard.md` §9.5 — the taste gap doesn't matter here |
| `apps/api/**`, `scripts/`, Terraform/infra | **Codex** | Backend/infra — no UI surface |
| QA/review across all of the above | **Claude** | See the GitHub Action below |

**If you are Codex implementing a mobile screen:** read
`design-system.md` first and build strictly against it — don't
improvise spacing, color, or component choices not covered there;
flag a gap in the PR description instead of guessing, so Claude's
review catches it as a design-system omission to fix once, not a
one-off judgment call to repeat.

**If you are Claude reviewing a mobile screen Codex built:** don't
just review the code — actually render the screen (or request a
screenshot in CI, see the GitHub Action) and check it against
`design-system.md` and the per-screen spec in `frontend-mobile.md`.
A screen that passes its tests but looks generic or inconsistent
with the system is a real review finding, not a nitpick.

**If you are Codex working anywhere in `apps/mobile`'s logic
layer:** stay inside hooks/services/store — don't restructure
screens or components even if it would simplify your logic; flag
it as a suggestion instead and let the screen's assigned
implementer (Codex-against-system, or Claude for the named
exceptions) make that call.

## Build & test commands

```
# API (FastAPI)
cd apps/api && pytest --cov=app
cd apps/api && ruff check . && mypy app

# Mobile (React Native, iOS + Android)
cd apps/mobile && npm test
cd apps/mobile && npm run lint && npm run type-check

# Dashboard (Next.js)
cd apps/dashboard && npm test
cd apps/dashboard && npm run lint && npm run type-check

# Full local stack
docker compose up
```

## Non-negotiable conventions

- Every feature PR references its User Story ID (`US-XX`) from
  `prd.md` in the PR description
- Data model changes require a numbered amendment in
  `data-model.md` (see §6.4/§6.5 for the pattern), not a silent edit
- API changes must keep `api-spec.md` in sync — CI fails on drift
  between the committed spec and FastAPI's generated OpenAPI output
- Check-in and SOS features (`US-15`, `US-16`) are safety-critical:
  any change touching the offline queue, sync retry logic, or
  notification dispatch requires the full test suite in
  `testing-qa.md` §14.4 (offline/chaos scenarios), not just unit
  tests, before merge

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [block-sig-hash/damdam](https://github.com/block-sig-hash/damdam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->

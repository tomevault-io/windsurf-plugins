---
trigger: always_on
description: > Loaded automatically by Claude Code at session start. Keep tight (~200 lines). For full architecture details see README.md; for refactor specs see `docs/superpowers/`.
---

# Claude Project Context — Ariyana Event Intelligence CRM

> Loaded automatically by Claude Code at session start. Keep tight (~200 lines). For full architecture details see README.md; for refactor specs see `docs/superpowers/`.

## Project

Event-intelligence CRM with React 19 + TS strict frontend (Vite), Express + Postgres backend (`api/`), and AI integrations (Gemini + Vertex + OpenAI). Solo dev. Active codebase optimization (sub-projects #1–#10) — see roadmap below.

## Conventions

- **Solo dev, commits straight to `main`.** No PRs, no review.
- **No CI yet (sub-project #7).** Pre-push hook runs `npm run typecheck && npm run typecheck:api && npm test` — that's the only gate. Never bypass with `--no-verify` unless explicitly authorized.
- **Pre-commit hook** runs `lint-staged` (ESLint --fix + Prettier on staged files). Blocks on lint errors.
- **No React Testing Library yet.** Component tests are deferred until a future "frontend test infra" sub-project. For UI changes, manual browser smoke is the only safety net — run after every commit, do not batch.
- **Pure helpers + custom hooks** are the testable layers in the frontend. Tests live alongside source: `*.test.ts`.
- **TS strict** is on except for `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` (deferred to sub-project #5 — see `STRICT_DEBT.md`).
- Daily scripts: `npm run lint | lint:fix | format | format:check | typecheck | typecheck:api | build | test | test:watch | test:coverage`.

## Where things live

- `STRICT_DEBT.md` — canonical tracker for `@ts-nocheck` / `@ts-expect-error` markers + deferred TS flags. Update its "Resolved" section when clearing markers.
- `docs/superpowers/specs/YYYY-MM-DD-<name>-design.md` — design specs (immutable once written).
- `docs/superpowers/plans/YYYY-MM-DD-<name>.md` — step-by-step implementation plans.
- `api/src/services/ai/` — AI provider abstraction (prompts in `prompts/`, scaffolded providers in `providers/`).
- `views/IntelligentDataView/EventModal/` — exemplar of refactor pattern (sub-project #4c done).

## Refactor roadmap status (as of 2026-05-07)

| Sub-project                                           | Status         | Notes                                                                                   |
| ----------------------------------------------------- | -------------- | --------------------------------------------------------------------------------------- |
| #1 Cleanup + tooling baseline                         | ✅ done        | Prettier, ESLint flat, TS strict, Zod env, husky                                        |
| #2 Test infra                                         | ✅ done        | Vitest, 183 tests across 20 files                                                       |
| #3 Structured logger / observability                  | pending        |                                                                                         |
| #4b Refactor `components/LeadDetail.tsx`              | 🔄 in progress | 1130 LOC `@ts-nocheck` (was 1998). Plan in 10 commits / ~5 sessions. Tasks 1–5/10 done. |
| #4c Refactor `views/.../EventModal.tsx`               | ✅ done        | Template for #4b. 7 sub-components + tested pure data fn.                               |
| #4d Refactor `views/LeadsView.tsx` (1914 LOC)         | pending        | Same playbook as #4b.                                                                   |
| #4 Refactor `api/src/routes/excelImport.ts` (973 LOC) | pending        | God file with 2 markers.                                                                |
| #5 API layer-ization + type normalization             | pending        | 5 markers remain (imap×2, excelImport×2 — also #4, managerReport).                      |
| #6 node-cron v4 migration                             | pending        | 1 marker (`scheduledReportsJob.ts`).                                                    |
| #7 GitHub Actions CI                                  | pending        | Replaces pre-push hook.                                                                 |
| #9 Bundle splitting / lazy loading                    | pending        |                                                                                         |
| #10 Documentation polish                              | pending        |                                                                                         |
| AI provider abstraction (out-of-band)                 | ✅ done        | Prompts extracted to `services/ai/prompts/`. Providers scaffolded.                      |

## Currently active: #4b LeadDetail refactor

- **Spec:** `docs/superpowers/specs/2026-05-07-lead-detail-refactor-design.md`
- **Plan:** `docs/superpowers/plans/2026-05-07-lead-detail-refactor.md` (10 tasks)
- **Approach:** Tab + custom hook decomposition (different from EventModal's pure-derive pattern because LeadDetail is genuinely stateful per-tab — 21 useStates split across 3 tab concerns).
- **Target structure:** `LeadDetail.tsx` <100 LOC orchestrator + `LeadDetail/{leadDetailHelpers, useLeadEdit, useLeadEnrichment, useLeadEmail, LeadInfoTab, LeadEnrichTab, LeadEmailTab}`.

### Done


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lanfurama/ariyana-event-intelligence-crm](https://github.com/lanfurama/ariyana-event-intelligence-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

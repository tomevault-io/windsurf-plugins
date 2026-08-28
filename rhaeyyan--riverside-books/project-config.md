---
trigger: always_on
description: **One file, two names.** Codex CLI reads `AGENTS.md` and Claude Code reads `CLAUDE.md`, so `CLAUDE.md` is a symlink to this file. Edit either path; there is only one document underneath, and no sync step. It used to be two hand-maintained copies that drifted to 96.4% identical — the same rule had to be fixed twice, and a fix that landed in one file silently missed the other.
---

# Riverside Books — Multi-Product Team Repo

**One file, two names.** Codex CLI reads `AGENTS.md` and Claude Code reads `CLAUDE.md`, so `CLAUDE.md` is a symlink to this file. Edit either path; there is only one document underneath, and no sync step. It used to be two hand-maintained copies that drifted to 96.4% identical — the same rule had to be fixed twice, and a fix that landed in one file silently missed the other.

Team build for the Cycle 4 "Direct-to-Consumer Retail" project brief (`docs/Cycle 4_ Project briefs.md`), shared across four collaborators in this one repo, each owning a product directory:

| Product | Directory | Owner |
| --- | --- | --- |
| A — Customer Ordering & Loyalty App | `product-a/` | [@rhaeyyan](https://github.com/rhaeyyan) |
| B — Staff Inventory & Ops Dashboard | `product-b/` | [@Cheewaiyip](https://github.com/Cheewaiyip) |
| C — Customer Support Chatbot (docs) | `product-c/` | [@humaali-create](https://github.com/humaali-create) |
| C — Customer Support Chatbot (app shell) | `product-c-app/` | [@humaali-create](https://github.com/humaali-create) |
| D — Marketing Content Generator | `product-d/` | [@crystalwatson-art](https://github.com/crystalwatson-art) |

This file is the standalone, canonical source for this repo's protocols — GitHub workflow, engineering standards, and the multi-agent build workflow below — so any teammate's agent session behaves the same way without cross-referencing a second document.

## Stack & docs

- **Next.js (App Router), TypeScript, Tailwind, Supabase (Postgres + Auth), deployed on Vercel.** One app per product, one shared Supabase project. Per-product reasoning lives in each product's own `tech_stack_recommendation.md` where one exists yet (not every product has written theirs — see Current scaffolding state below) — don't re-derive a stack decision already made there.
- **`docs/PRD.md` is the whole-suite requirements source of truth** — problem statement, user stories (P0/P1/P2), success metrics, technical requirements, out-of-scope, and the live risk/blocker log (Section 7). Read it before starting non-trivial work on any product; it's assembled from all four products' docs and is the fastest way to see how a change in one product affects the others.
- **`docs/schema.md` is the single shared table contract.** Product A owns and migrates every table listed there; other products read it rather than restating field lists locally — restating is exactly how this repo's schema drifted before that file existed (three independently-invented `events` shapes at one point). If a task needs a new or changed shared table, that's a cross-team decision, not something to make inline — see the Directory boundary rule below.
- **`docs/assumptions.md`** records the store's stated operating assumptions (no POS exists, staffing, catalog size, reconciliation cadence) once, for all four products to build against — the store is fictional, so these are stated rather than researched.
- **`docs/model-access.md`** is the shared LLM-access research for Products C and D: which products need a model (only C and D, and narrowly), cost, latency, provider options, and the grounding/fact-protection architecture both products build against. Read this before writing model-calling code in either product.
- **Per-product build plans**: each product's own `implementation_plan.md` is phased, with exit conditions per phase. Nothing after a product's Phase 0 is allowed to break that product's deployment.
- **Current scaffolding state**: only `product-c-app/` has a real Next.js app as of this writing. `product-a/`, `product-b/`, and `product-d/` have docs (`market_strategy.md`, `implementation_plan.md`, and `tech_stack_recommendation.md` for A and B — Product D hasn't written one yet) but no scaffolded app — their Phase 0 is the next unit of work in each case.

## Commands

Each product app is its own npm project — run these from inside `product-c-app/` today, and from inside the equivalent `product-X-app/` (or `product-a/`, if that ends up unprefixed) once the other three scaffold theirs. There is no root-level `package.json`.

- `npm run dev` — local dev server. `npm run build` / `npm run start` — production build/serve.
- `npm run lint` — app code (ESLint). There is **no `format`/`format:check` script** in `product-c-app` — `.prettierrc.json` exists at the repo root, but nothing currently runs it (see the markdown bullet below).
- `npm run typecheck` — Vitest and ESLint do **not** type-check; a type error in a test file is invisible without this. For a Next.js app, run `next typegen` first (or as part of the script) — a bare `tsc --noEmit` can't see Next's generated route/layout types otherwise.
- `npm run test` — Vitest (`vitest run`). **No coverage is collected** — no `--coverage` flag, no `coverage` block in `vitest.config.ts`, and no `@vitest/coverage-v8` installed — so nothing in this repo can report a coverage figure today.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhaeyyan/riverside-books](https://github.com/rhaeyyan/riverside-books) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

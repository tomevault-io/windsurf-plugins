---
trigger: always_on
description: sur9e is a free, self-hosted, open-source job-hunt toolkit that runs inside
---

# sur9e — AI job-hunt command center

sur9e is a free, self-hosted, open-source job-hunt toolkit that runs inside
your AI coding agent (Claude Code, Codex, or OpenCode) and ships
a local web UI on top of the same data. It evaluates job offers against your
real career profile, screens cheap before evaluating deep, tailors CVs, and
tracks every application — all on your machine.

**Mission:** quality over quantity. AI gives the job-seeker velocity and
clarity, never shortcuts — sur9e will never auto-submit an application.

This file is the **operating manual for the AI agent** (the agent IS the CLI —
read this on every session). The detailed human-and-agent contribution workflow
lives in [`CONTRIBUTING.md`](CONTRIBUTING.md). This operating manual is stored as
both `CLAUDE.md` and `AGENTS.md`; keep those files byte-identical.

**Honesty rule:** never hallucinate. If unsure, state uncertainty. Say "I don't
know" rather than guess.

## Source of truth

| Concern                        | File                                                              |
| ------------------------------ | ----------------------------------------------------------------- |
| Data contract (User vs System) | [`docs/data-contract.md`](docs/data-contract.md)                  |
| First-run onboarding           | [`docs/onboarding.md`](docs/onboarding.md)                        |
| Architecture (system flow)     | [`docs/architecture.md`](docs/architecture.md)                    |
| Setup & prerequisites          | [`docs/setup.md`](docs/setup.md)                                  |
| Personalization guide          | [`docs/customization.md`](docs/customization.md)                  |
| Contribution workflow          | [`CONTRIBUTING.md`](CONTRIBUTING.md)                              |
| Releases & repo automation     | [`docs/releasing.md`](docs/releasing.md)                          |
| Bugs / feature requests        | GitHub Issues in this repo                                        |
| Your CV                        | `inputs/personalization/cv.md` (gitignored)                       |
| Your profile & targets         | `inputs/personalization/profile.yml` (gitignored)                 |
| Your archetypes & narrative    | `inputs/personalization/narrative.md` (gitignored)                |
| Your proof points              | `inputs/personalization/article-digest.md` (gitignored, optional) |
| Your ATS portals               | `inputs/personalization/portals.yml` (gitignored, optional)       |

## Session start

1. Run silent update check: `node update-system.mjs check`. If
   `update-available`, surface to the user (see Update check protocol below).
2. If required user files are missing (`inputs/personalization/cv.md`,
   `inputs/personalization/profile.yml`) → enter onboarding
   (see [`docs/onboarding.md`](docs/onboarding.md)).
3. **Wizard handshake:** if the first user message is `Set me for success, baby!`
   — the playful line `npm run setup` seeds on hand-off — match its energy, then
   run onboarding ([`docs/onboarding.md`](docs/onboarding.md)). It's the wizard's
   launch signal, not a normal request.

## Architecture

Next.js 16 (App Router, Turbopack) + React 19 frontend, Server Actions for
mutations, with a thin Node-only library layer underneath. Detail in
[`docs/architecture.md`](docs/architecture.md).

```
src/app/                  — Next.js App Router (routes + RSC pages + /api/* JSON compat)
src/features/<feature>/   — Feature-folder UI (profile, report, table, pipeline, analytics, settings)
src/components/primitives — Button, Input, Select, Card, Pill, Chip, Field, etc. (Radix-backed)
src/components/domain/    — StatusPill, ScoreChip, ActionsMenu (composed primitives)
src/components/modals/    — Apply, Screen, Evaluate, Followup, CV, CoverLetter, Research, Outreach
src/components/shell/     — Topbar, Rail, mobile-nav, chrome-effects
src/server/actions/       — Server Actions (applications, profile, settings, jobs)
src/server/revalidate.ts  — Type-safe wrapper around Next's revalidatePath
src/lib/server/           — Node-only loaders / writers / schemas (applications, profile, settings, reports, pipeline, usage, jobs)
src/lib/schemas/          — zod schemas shared by client + server
src/lib/api/              — fetchJson helper + tiny client/server bridges
src/lib/forms/            — useZodForm (rhf + zodResolver wrapper)
src/hooks/                — TanStack Query wrappers, useFocusTrap, useJobAction
src/stores/               — Zustand stores (drawer, selection, modal, toast, status-popover, etc.)
src/app/styles/           — Global CSS (tokens.css is the single source of truth for design tokens)
src/proxy.ts              — Next 16 proxy (was middleware.ts; no-op pass-through today)
inputs/personalization/   — User CV / profile / narrative / digest (gitignored)
inputs/config/            — Settings (gitignored)
content/modes/            — Agent mode prompts (one per evaluation type)
content/templates/        — PDF / CV / state templates
content/examples/         — Personalization templates new users copy from
cli/                      — Node CLI tools (doctor, verify-pipeline, generate-pdf, merge-tracker, etc.)
scripts/                  — Web launcher, setup migrations, maintainer tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arspesk/sur9e](https://github.com/arspesk/sur9e) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->

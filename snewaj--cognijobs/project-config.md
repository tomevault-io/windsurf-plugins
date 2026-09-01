---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Current status (read this first)

**The app has been built.** This repository started as a pure documentation/handover package for "Bdjobs", an
employment platform — but the build described below has since been completed end to end and the app was renamed
**Bdjobs → CogniJobs** (commit `60bf04c`). There is now real source code:

- **`backend/`** — the .NET modular monolith (12 bounded-context modules), built from `Handover_Packages/BC-*.md`.
- **`frontend/`** — the Next.js 4-portal app, built from `Handover_Packages/FA-*.md`.
- **`infra/`** — AWS EC2 deployment scripts (Docker Compose + nginx on a single host).

All 18 units of the build pipeline described below are `done` (`.orchestration/STATUS.md`), with full backend/
frontend test suites green as of the last pass (see `HANDOVER.md`). Current work has shifted to deployment, not
app-building — **`HANDOVER.md` (repo root) is the up-to-date session handover; read it first for what's actually in
flight.** `BUILD_REPORT.md` is the full historical build log if you need narrative detail on how a given module/portal
was built or what bugs were fixed along the way.

The rest of this file (below) describes the **original spec** that drove the build. It's kept as design-rationale
reference — useful if you're extending the app and want to understand *why* something was built a certain way — but
it is not a from-scratch build brief anymore. Note also that `Handover_Packages/`, `Stories/`, and `SRS_Sections/`
are **not tracked in git** (see root `.gitignore`) — they exist on disk locally but aren't part of the shipped repo.
The spec still says "Bdjobs" throughout (expected — it predates the rename and wasn't updated, since it's frozen
reference material, not shipped documentation).

If asked to **extend or modify the app**, read the existing code in `backend/`/`frontend/` first — that's the
ground truth now, not the spec. Cross-reference the relevant `Handover_Packages/BC-*.md` or `FA-*.md` file only when
you need the original design rationale (contracts, acceptance criteria) behind existing behavior.

## Original spec structure (historical — see above)

There are three top-level folders that made up the original handover package:

- **`Handover_Packages/`** — the build briefs.
- **`Stories/`** — supporting artifacts: user stories, DDD strategic design (context map, event catalog), and UI traceability (screen catalog, story↔screen map).
- **`SRS_Sections/`** — the underlying Software Requirements Specification, split into one file per section (also an Obsidian vault — `SRS_Sections/.obsidian/`). This is the source requirements; `Handover_Packages/` and `Stories/` are derived from it for build purposes.

## Entry points (read in this order) — for understanding the original spec/design rationale

1. **`Handover_Packages/BUILD_ORCHESTRATION.md`** — the master orchestrator brief. Describes the program as an **18-step sequential, single-worker-at-a-time** pipeline (backend foundation → 12 backend modules in dependency waves → frontend foundation → 4 frontend portals), with a durable checkpoint/resume protocol under `/.orchestration/` so work survives context limits. **All 18 units are now `done` per `.orchestration/STATUS.md`** — this checkpoint/resume protocol is no longer live; treat `STATUS.md` as a historical record of build order, not a resume point, unless the app itself needs a from-scratch rebuild. (Originally 19 steps/5 portals — step 19/FA-4 Partner was dropped when FA-4 merged into FA-3; see below.)
   - **Default to this sequential mode, not parallel multi-agent fan-out.** §1/§4 are explicit: "one worker at a time", "No two workers run at once" — spawn one worker for the next unit, wait for it to finish and merge, checkpoint, then advance. The "running modules/portals in parallel" sections in the two track guides (§5 of each) are an **optional**, opt-in alternative for fanning out within a wave/phase — they are not what the master orchestrator does by default, and should only be used if explicitly requested.
2. **`Handover_Packages/BACKEND_BUILD_INSTRUCTIONS.md`** — entry point for backend work (the 12-module modular monolith).
3. **`Handover_Packages/CLAUDE_CODE_BUILD_INSTRUCTIONS.md`** — entry point for frontend work (the 4-portal web app).
4. **`Handover_Packages/00-Shared-Foundations.md`** — cross-cutting backend brief: the (initially blank) Target Stack declaration, neutral type/notation vocabulary, 5-layer module structure, shared-kernel building blocks (`Entity<Id>`, `AggregateRoot<Id>`, `Result`/`Result<T>`, `Error`), outbox/inbox conventions, testing strategy. Every `BC-*` package assumes this.
5. **`Handover_Packages/00-Frontend-Foundations.md`** — cross-cutting frontend brief: target stack, design system, app shell, routing/guards, API client, `Error.Code` → UI mapping, i18n, accessibility, testing. Every `FA-*` package assumes this.
6. **`Stories/Context_Map.md`** — DDD strategic design showing how the 12 bounded contexts relate (OHS/PL, ACL, Customer/Supplier, Partnership, Conformist). Determines backend build/parallelization order.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snewaj/CogniJobs](https://github.com/snewaj/CogniJobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->

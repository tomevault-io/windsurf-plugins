---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Sentra — a mobile **personal health-management app** ("personal lab-result archive + checkup planner") for the EU + Ukraine markets, mobile-first. The full design/spec chain (SDLC 01→03.5) is done for all 7 MVP features, and the **MVP is now implemented**: the `server/` .NET modular monolith (F0–F6 backends behind ports-and-adapters, EF+Postgres, integration tests) and the `mobile/` Flutter app (all six MVP feature UIs) are built. Product documentation lives under `docs/` alongside the Penpot screen designs. The raw founder note `ideas.md` lives at the repo root; the maintained product index is [[BACKLOG]] under `docs/`. See [[READING-GUIDE]] for how the docs chain together, and the **Working notes** section below for how to build/run/test the code.

**This is a clean greenfield.** Nothing is shipped, there are no users, no data, and no migrations. Don't worry about historical data, backward compatibility, deprecation paths, or migrating anything — design for the right end state directly. If a file or decision turns out wrong, just change or delete it. (The only "history" is `ideas.md`, kept purely as the origin note — not a constraint.)

**Solo project — one person wears every hat.** The owner/PM/designer/developer is the same person: **Viacheslav Melnichenko**, a .NET backend developer (~8 years, microservices). So:
- Role labels in docs (`owner: PM`, `owner: Tech Lead`, `owner: UX`, "medical advisor") are **hats, not people** — they all currently resolve to Viacheslav. They mark *which mindset* a task needs, not a hand-off to someone else. Don't treat them as a team or invent stakeholders.
- Backend/architecture calls can lean on real .NET + microservices expertise — be concrete and senior there, not hand-holding.
- There is no separate budget, legal team, or medical staff yet. When a task implies one (lawyer sign-off, medical advisor), treat it as "Viacheslav to source later" — a flagged dependency, not a blocker to design around. Prefer approaches a solo founder can actually execute now (curate from public sources + disclaimer) over ones that need hiring.

## Documentation language policy (IMPORTANT)

**English is the single source of truth for all documentation.** Every document is authored and maintained in English.

- `<doc>.md` — **English. The source of truth.** When anything changes, change this file.
- `<doc>.uk.md` — **Ukrainian. Reference/comprehension copy only.** It exists so Ukrainian-speaking stakeholders can read along. It is **never** the authority and **never** the basis for downstream work (PRD, architecture, ADR, code).

Rules:
- **Ukrainian (`.uk.md`) lives only under `docs/`.** No other directory (`src/`, `app/`, config, scripts, tests, etc.) may contain Ukrainian — code, comments, identifiers, commit-adjacent files, and all non-`docs/` content are English-only. Ukrainian is a documentation-reading aid, nothing more.
- When you create or edit any English doc under `docs/`, update its `.uk.md` sibling in the same commit so they do not drift. If you cannot, note the `.uk.md` as stale rather than leaving a silent mismatch.
- **Exception — deep-technical artifacts (SAD / `sad.md`, **every** ADR under `adr/`, and tech indexes `ARCHITECTURE-OPEN-DECISIONS`, `IMPLEMENTATION-READINESS`):** each carries a short `.uk.md` **summary** (the decision + why, not a full translation). This is per-ADR — a new ADR means a new `<adr>.uk.md` in the same commit. The English file is the only authority; the `.uk` summary is a reading aid. Product docs (idea-brief, PRD, CONTEXT, BACKLOG, DECISION-LOG) keep full `.uk.md` copies as before.
- **Exception — implementation artifacts (`data-model.md`, `contracts/openapi.yaml`, `tasks/`, `test-plan.md`):** these are code-adjacent and stay **English-only**. Instead of per-file `.uk` siblings, each feature has **one** `overview.uk.md` — a compact Ukrainian reading aid covering data-model + API + tasks + tests together. The English files are the only authority; never derive anything from `overview.uk.md`.
- **Exception — design artifacts.** Per-feature `design-spec.md` + `design-sync-report.md` are design-adjacent and stay **English-only** (covered by the feature's `overview.uk.md`). Cross-cutting design docs under `docs/design-system/` follow the deep-technical rule: substantive ones (`NAVIGATION-MAP.md`, `UX-REVIEW.md`) carry a short `.uk.md` **summary**; pure indexes/data (`README.md`, `color-tokens.json`, `brand/`) are English-only. The top-level `TRACEABILITY-AUDIT.md` keeps a short `.uk.md` summary like the other tech indexes.
- Never derive requirements, decisions, or RICE/feasibility values from a `.uk.md` file. If the English and Ukrainian versions disagree, **the English `.md` wins** — fix the `.uk.md`.
- The root file `ideas.md` is the original founder note (mixed Ukrainian/English) and is exempt — historical input, not maintained documentation.
- Verbatim user quotes inside a doc (e.g. the "Raw idea" section) may be paraphrased into English in the `.md`; the original-language wording is preserved in the `.uk.md`.

## Markdown style — Obsidian (IMPORTANT)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HeliosPersonal/sentra](https://github.com/HeliosPersonal/sentra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

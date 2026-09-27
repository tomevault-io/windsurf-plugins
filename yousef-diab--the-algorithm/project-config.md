---
trigger: always_on
description: This file applies to the entire repository. Read it before editing. More specific `AGENTS.md` files, if added later, override it only within their directory.
---

# AGENTS.md — working guide for The Algorithm

This file applies to the entire repository. Read it before editing. More specific `AGENTS.md` files, if added later, override it only within their directory.

The active product is a Next.js learning application backed by Postgres. The repository also retains the original static course sources and generator as an import source and historical offline build. Do not confuse the two architectures.

## 1. The rule that overrides everything

**Course content must come purely from the provided ICT mentorship notes and video transcripts.** Do not add outside trading knowledge, invent examples, or silently “improve” concepts beyond what the source supports.

- Read the relevant file under `transcripts/` and/or `notes/` before drafting lesson prose, quiz answers, summaries, or explanations.
- Correct answers and explanations must be traceable to those sources. Plausible distractors may be invented, but they must not introduce false teaching.
- If the sources are ambiguous or incomplete, under-claim and report the gap.
- Preserve attribution to ICT and the original creators.
- `transcripts/` and `notes/` are local, git-ignored source material. Never commit them.

Product copy, UI labels, infrastructure, and engineering documentation are not course content and may be written normally.

## 2. Current architecture

### Active application

- `app/` — Next.js 16 App Router routes, server actions, route handlers, layouts, and page-level styles.
- `components/` — client and server UI grouped by concern: auth, lessons, quizzes, progress, rewards, shell, notes, and lightbox.
- `components/shell/SiteLoader.tsx` — the shared visual loader for route boundaries. Reuse it instead of creating route-specific loading cards or spinners.
- `lib/db/schema.ts` — Drizzle schema for application-owned tables in Postgres.
- `lib/db/` — authenticated per-user queries for access, progress, quizzes, exams, and notes.
- `lib/content/` — block validation/rendering, public reads, imports, draft writes, canonicalization, and admin queries.
- `lib/rewards/` — checkpoint XP, leaderboard queries, levels, and focus-timer state.
- `lib/auth*` — Neon Auth server/client integration. Auth users live in `neon_auth."user"`; application tables store their IDs as text.
- `drizzle/` — append-only SQL migrations and Drizzle snapshots. Migrations `0007`–`0009` implement rewards and rollout reconciliation.
- `mcp/` — the local content-authoring MCP server. It can create drafts and edit permitted metadata; it cannot publish.
- `scripts/` — deliberate operational CLIs for imports, media, access, status, draft promotion, entitlement grants, and database checks.
- `tests/unit/` — default, isolated Vitest suite. Reward SQL runs against disposable PGlite.
- `tests/browser/` — isolated reward-component browser fixture.
- `tests/e2e/` — Playwright tests against a built Next.js app; authenticated variants may use configured accounts and data.
- `tests/integration/` — tests against the configured real database. They are intentionally separated from the default Vitest configuration.

The active deployment reads course content from Postgres. Public catalog metadata may be cached, while gated bodies and media must only be fetched after authorization.

### Retained static source tree

- `content/` contains the original per-section, per-month/part, per-lesson HTML and quiz source.
- `engine/` contains the original static renderer.
- `build.py` assembles those sources into `index.html`.
- `index.html` is generated. Never hand-edit it.
- `verify.py` rebuilds and checks the static artifact.

The static tree remains useful for bulk import, provenance, and the legacy offline artifact. It is no longer the architecture of the active Next.js application.

## 3. Content and data conventions

### Course shape

The corpus currently has two sections and 78 teaching lessons:

- ICT Core: Months 1–4, 38 lessons.
- ICT 2022 Mentorship: Parts 1–6, 40 lessons; episode 28 is omitted because it has no usable teaching audio/source.

Sections may also have a review page and final exam. Database lesson kinds are `lesson`, `review`, and `exam`. Access is `free`, `members`, or `admin`; status is `draft` or `published`. Unknown access/status values must fail closed.

Lesson IDs and slugs are durable identifiers. Do not rename them casually: they connect content, media, progress, answers, rewards, caches, and URLs. Quiz results key on stable question UUIDs, never question order.

### Quizzes

- Keep four options per question.
- Options shuffle at render time; author the correct `answer` as the zero-based index in stored order.
- Keep option lengths reasonably balanced so the correct answer is not visually obvious.
- Put nuance and source-grounded explanation in the explanation field rather than making one option conspicuously detailed.
- Reordering a question must preserve its UUID. Rewording/deleting questions can invalidate saved answers and must be treated as a data-affecting change.

### Database content and drafts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yousef-Diab/the-algorithm](https://github.com/Yousef-Diab/the-algorithm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

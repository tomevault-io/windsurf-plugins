---
trigger: always_on
description: Repository contract for soundspan.
---

# AGENTS.md

Repository contract for soundspan.

## Quick Start

1. Read this file for repo rules and conventions.
2. See [CONTRIBUTING.md](CONTRIBUTING.md) for build, test, and PR workflow.
3. If your agent runtime provides AWM, see [.awm/AGENTS-AWM.md](.awm/AGENTS-AWM.md) for the enhanced workflow. If you are unaware or unsure of what AWM is, do not read the file.
4. If using Claude, also read [CLAUDE.md](CLAUDE.md).

## Source Of Truth

- Follow this file first.
- `CLAUDE.md` and `.claude/awm-broker/**` are tool-specific companions. If they disagree with this file, this file wins.

## Coding Standards & Handbook

- The canonical external engineering-standards baseline for soundspan is the [coding-handbook](https://github.com/BonzTM/coding-handbook) repo (local sibling checkout at `~/git/coding-handbook` when available).
- All contributors and AI agents — including any tools they drive (e.g. codex) — must follow the handbook's language module for the code they touch:
    - `typescript/` for TS/JS (backend, frontend, packages)
    - `python/` for the `services/**` sidecars
- Precedence, so this composes with the rules below:
    1. This AGENTS.md and its repository-specific rules win.
    2. Then the coding-handbook language module.
    3. Then general industry best practice.

    Handbook guidance that conflicts with an explicit rule in this file defers to this file.

- The handbook is the baseline the codebase's enterprise-standards refactor was aligned to; new work must not regress below it.

## Working Rules

- **Read before edit.** Read the full relevant source before making changes. Do not guess at file contents or structure.
- **Smallest safe change.** Make the minimum change that solves the problem. Preserve existing style and conventions. Do not refactor adjacent code, add unsolicited features, or "improve" what wasn't asked for.
- **TDD for executable changes.** For code, schema, or behavior changes, write or update a failing test first, then implement until it passes. Deviations require explicit user approval. Non-executable work (docs, config review, planning, workflow governance) is exempt.
- **No invented requirements.** Do not invent product requirements, compatibility guarantees, or migration behavior when the repo does not define them. Surface the decision and wait for direction.
- **Targeted testing only.** Do not run the full test suite — it maxes out available RAM. Run only the test files and suites relevant to the current changes.
- **Prefer small, reviewable changes** over broad cleanup.

## Repository-Specific Rules

- **API boundary:** Use `frontend/lib/api.ts` as the frontend API boundary. No direct `fetch` calls from components.
- **Backend config:** Read env through `backend/src/config.ts`.
- **Database access:** Prefer Prisma for all DB access. Raw SQL (`$queryRaw`/`$executeRaw`) is permitted **only** for the classes of query Prisma cannot express, namely:
    - **pgvector similarity / ANN** — ivfflat probe tuning and `<=>`/`<->` distance ordering over embedding columns (e.g. `backend/src/utils/annQuery.ts`, `backend/src/services/trackEmbeddings.ts`, `backend/src/services/hybridSimilarity.ts`).
    - **PostgreSQL full-text search** — `tsvector`/`to_tsquery`/`ts_rank` ranking (e.g. `backend/src/services/search.ts`).
    - **Row-level & advisory locking** — `FOR UPDATE SKIP LOCKED` job claiming and `pg_advisory_*` locks (e.g. `backend/src/routes/downloads.ts`, worker claim loops).

    Constraints on any permitted raw SQL: use Prisma tagged-template `$queryRaw`/`$executeRaw` so every value is bound as a parameter; **never** `$queryRawUnsafe`/`$executeRawUnsafe` with interpolated external input (dynamic identifiers must come from code-owned allowlists); back it with a behavioral test against real PostgreSQL, not a source-text assertion (see Testing below). Anything a Prisma query can express — plain filters, counts, existence checks — must use Prisma, not raw SQL.

- **Logging helpers:** Use shared logging helpers in runtime code, and scope logs with `logger.child("Scope")` rather than ad hoc `[bracket-tag]` message prefixes so scope is a structured field:
    - frontend: `frontend/lib/logger.ts`
    - backend: `backend/src/utils/logger.ts`
    - python sidecars: `services/common/logging_utils.py`
- **Naming & placement conventions:** `camelCase` TypeScript source files; one route module per mounted prefix under `backend/src/routes/` (indexed in `backend/src/routes/README.md`); frontend domain modules live under `frontend/features/<domain>/` and are indexed in `frontend/features/README.md`; scheduled/background processors belong in `backend/src/workers/`. Known drift (colocated vs. tree tests, `components/vibe` placement) is documented in the relevant README rather than silently tolerated — follow the README when extending an area.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soundspan/soundspan](https://github.com/soundspan/soundspan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->

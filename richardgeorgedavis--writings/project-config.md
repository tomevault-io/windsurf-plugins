---
trigger: always_on
description: Purpose: provide concise, actionable guidance for future Copilot/Codex sessions working in this repository.
---

# Copilot instructions for Richard George Davis — writings repo

Purpose: provide concise, actionable guidance for future Copilot/Codex sessions working in this repository.

1) Build, test, and lint commands

- Install: npm install
- Dev server: npm run dev
- Production build: npm run build
- Serve production locally: npm run start
- Serve local reference mirror: npm run serve:ref
- Lint: npm run lint
- Typecheck: npm run typecheck
- Verify (pre-release check: typecheck + lint + build): npm run verify
- Snapshot freshness check: npm run check:snapshots
- Snapshot/content sync: npm run sync:data (legacy alias: sync:cache)
- Rebuild static client search assets: npm run sync:static-assets
- Re-extract local ref manifest: npm run sync:ref-manifest

Focused tests use Node's built-in test runner through `tsx --test`, for example `npx tsx --test lib/production-env.test.ts lib/discussions.test.ts`. Access-specific coverage is available through `npm run test:commerce-access`.

2) High-level architecture (big picture)

- Framework: Next.js (next v16) + React (v19) TypeScript-based app. Styling uses Tailwind/PostCSS.
- Content model: snapshot-driven. Live sources (Google Sheet for Poems/Stories and Substack RSS for Articles) are fetched by operator-only sync scripts and written to data/writing.cache.json. Runtime pages read from that snapshot, keeping the app static-first.
- Routing: canonical site routes include /, /category/[slug], /article/[slug], /resources, /docs, /docs/[slug], /docs/template-components, /docs/template-components/[origin], /docs/template-components/[origin]/[source], and /legal/[slug]. Legacy compatibility routes remain.
- Operations: sync scripts (scripts/*.ts) populate the cache. POST /api/sync and POST /api/revalidate are available but protected by secrets. Reader responses and discussions are SQLite-backed writable state; discussion comments require Turnstile plus magic-link verification before public display.
- Build & release: production builds use the committed snapshot (data/writing.cache.json). A static export is an allowed release option but the pipeline expects snapshot semantics.

3) Key conventions and repo-specific patterns

- Snapshot-first: do not rely on live fetches at runtime. Any content-change workflow should update the snapshot (sync:data) and commit the resulting data/writing.cache.json for it to be live.
- Source truth mapping:
  - Poems tab in the Google Sheet -> Poems
  - Stories tab in the Google Sheet -> Stories
  - Substack RSS -> Articles (enriched at snapshot time)
- Scripts namespace: commands with sync:* manage source sync, cache, and static assets. Respect these scripts for content operations rather than ad-hoc fetches.
- Secrets/environment contract: env vars used by ops:
  - GOOGLE_SHEETS_SOURCE_URL (only for live sync)
  - SUBSTACK_FEED_URL (optional)
  - SITE_URL (used for canonical URLs; set for production)
  - SYNC_SECRET and REVALIDATE_SECRET (protect POST endpoints)
  - READER_RESPONSE_SECRET
  - READER_RESPONSES_DB_PATH and DISCUSSIONS_DB_PATH (writable SQLite paths)
  - NEXT_PUBLIC_TURNSTILE_SITE_KEY and TURNSTILE_SECRET_KEY (discussion spam gate)
  - SMTP_* vars (discussion magic-link delivery, and later access-email delivery)

- Ref-first rule: the repo contains local reference shells in ref/. AGENT.md instructs to treat helio-1k2.pages.dev and ref/usegately.com as the live visual reference. Align shell and layout to those references.

- AGENT.md workflow rules (important for AI/automation):
  - AGENT.md is a repo-specific briefing file for humans and agent operators in this repository. Do not assume GitHub Copilot automatically includes AGENT.md in its prompt; open it explicitly when you need the fuller workflow and batch-order guidance.
  - When following AGENT.md in Codex or other agentic sessions, produce a Pre-Run Brief before making code changes. The brief must include model recommendation, task classification, dependency-ordered batch plan, assumptions, stop conditions, and desired run mode (RUN SAFE / RUN FULL / RUN UI FIRST / RUN CONTENT FIRST).
  - If you are working in a GitHub Copilot context that only auto-applies repository custom instructions, treat AGENT.md and HANDOVER.md as additional repo guidance to inspect, not as guaranteed auto-loaded instructions.

4) Where to look first (docs and handover)

- In GitHub Copilot, this `.github/copilot-instructions.md` file is the repository-wide custom instruction file that is auto-applied when supported. For deeper repo context, then open AGENT.md, README.md, docs/ (docs/README.md, docs/spec/*, docs/brief/*), and HANDOVER.md.

5) AI-assistant / other tooling configs

- Repository-wide GitHub Copilot custom instructions live in this file: `.github/copilot-instructions.md`.
- No `.github/instructions/*.instructions.md` path-specific Copilot instruction files were found.
- No CLAUDE.md, AIDER_CONVENTIONS.md, .cursorrules, or .windsurfrules files were found in the repo root.
- AGENT.md exists as additional repo guidance, but it is not the documented GitHub Copilot repository custom-instructions filename. Open it explicitly when you need the repo’s deeper batch workflow and operating rules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RichardGeorgeDavis/writings](https://github.com/RichardGeorgeDavis/writings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->

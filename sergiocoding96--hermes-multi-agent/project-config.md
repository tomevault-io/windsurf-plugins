---
trigger: always_on
description: **Single-tier memory architecture: MemOS only. Holographic Tier 1 deprecated 2026-04-28. v1 (MemOS server) remains the production target.**
---

# Hermes Multi-Agent Research System

## 🚨 Active sprint — read this first

**Single-tier memory architecture: MemOS only. Holographic Tier 1 deprecated 2026-04-28. v1 (MemOS server) remains the production target.**

Any new agent working in this repo should read these before acting:

1. **Decision doc (current):** [`memos-setup/learnings/2026-04-28-collapse-to-single-tier-memos.md`](memos-setup/learnings/2026-04-28-collapse-to-single-tier-memos.md) — why holographic was deprecated, what's gone, what's left, rollback path
2. **Previous direction (still relevant context):** [`memos-setup/learnings/2026-04-27-v2-deprecated-revert-to-v1.md`](memos-setup/learnings/2026-04-27-v2-deprecated-revert-to-v1.md) — why v2 was deprecated, what was fixed in v1
3. **MVP-readiness brief:** [`tests/v1/reports/combined/v1-mvp-readiness-2026-04-26.pdf`](tests/v1/reports/combined/v1-mvp-readiness-2026-04-26.pdf) — pre-fix audit + remediation plan
4. **Two-repo team explainer:** [`docs/architecture/two-repos.pdf`](docs/architecture/two-repos.pdf) — how Hermes (this repo) and MemOS (`sergiocoding96/MemOS`, your fork) fit together
5. **Operator runbook:** [`tests/v1/STEP-BY-STEP.md`](tests/v1/STEP-BY-STEP.md) and [`tests/v1/CC-PROMPTS.md`](tests/v1/CC-PROMPTS.md) — phase-by-phase commands for fix → re-audit → ship
6. **Sprint 1 history (still relevant):** [`memos-setup/learnings/2026-04-20-sprint-merge-log.md`](memos-setup/learnings/2026-04-20-sprint-merge-log.md) — what was shipped in the original v1 hardening sprint
7. **Superseded — historical only:** [`memos-setup/learnings/2026-04-20-v2-migration-plan.md`](memos-setup/learnings/2026-04-20-v2-migration-plan.md) (the original v2 migration plan)

If you are working inside a **worktree** under `~/Coding/Hermes-wt/` or `~/Coding/MemOS-wt/`, read the `TASK.md` in that directory — it's your full brief.

**Architecture status (2026-04-28):** Single-tier MemOS. Two-tier holographic+MemOS design was aspirational — Tier 1 (`holographic`) had zero rows in every profile after weeks of operation. Collapsed to one stack: agents read/write MemOS via `memos-toolset`, plus the always-on built-in memory layer in hermes-agent core. v2 plugin (`@memtensor/memos-local-plugin`) remains deprecated.

**Sprint 2 status (2026-04-27):** v2 audit failed (mean 2.4/10, min 1/10). v1 audit (clean re-run) found a fixable system at mean 5.2/10 with five surgical bugs. All five fixed across 6 PRs (Hermes #14/#15/#16, MemOS #6/#7/#8). v2 stays as a dormant spike; do not enable in production.

## Working Rules
- **ALWAYS use parallel agents for independent tasks.** When multiple fixes, tests, or investigations can run simultaneously, launch them all in one message. Never serialize work that can be parallelized.
- **ALWAYS read entire documentation before creating skills or integrations.** Use Firecrawl (localhost:3002) if WebFetch struggles with JS-rendered docs. Never create a skill based on partial information.
- When given a docs URL, scrape every page. Use Firecrawl's `/v1/scrape` endpoint for JS-heavy sites.
- **CLAUDE.md is the canonical sprint state — keep it current in the same PR that changes direction.** Any PR that changes the project's strategic direction (new sprint kickoff, deprecating a product, switching backends, audit results that overturn a prior plan, major architectural decision) MUST in the same PR:
  1. Update the "🚨 Active sprint" header at the top of this file to reflect the new direction
  2. Add a decision doc at `memos-setup/learnings/<YYYY-MM-DD>-<topic>.md` capturing the why
  3. Cross-link the two

  **Reviewers:** block PRs that change strategic direction without these updates.

  **Agents starting a fresh session:** before doing anything else, spot-check that the "🚨 Active sprint" header matches the most recent decision doc in `memos-setup/learnings/` (sort by date) and the most recent strategic merge commits on `main`. If the header is stale, flag it to the operator and propose an update before continuing the requested task. Stale strategic context is the failure mode this rule exists to prevent.

## What This Is
Layered multi-agent system: CEO (Claude Opus 4.6 via Paperclip) orchestrates specialized Hermes agents, each with isolated MemOS memory cubes. Two feedback loops: soft (user feedback → skill patches) and hard (Karpathy autoresearch-style metric threshold → auto-patch → re-run).

## Architecture
- **CEO Agent**: Claude Opus 4.6 on Paperclip (http://tower.taila4a33f.ts.net:3100)
- **Worker Agents**: Hermes (MiniMax M2.7) spawned via hermes-paperclip-adapter
- **Memory**: MemOS (Qdrant + Neo4j + SQLite) at localhost:8001 — single-tier; per-profile `memory.provider: ''` (no external Tier 1 plugin)
- **Web search**: Firecrawl (localhost:3002) → SearXNG (localhost:8888) — free, unlimited, aggregates Google+Bing+DDG+Startpage
- **Web scraping**: Firecrawl (localhost:3002) with Playwright service for JS-rendered pages
- **Anti-bot browser**: Camofox (localhost:9377) — Camoufox Firefox fork with C++ fingerprint spoofing, bypasses Cloudflare/anti-bot
- **Token burn rule**: Agents communicate ONLY via MemOS shared state, never agent-to-agent

## Key Paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sergiocoding96/hermes-multi-agent](https://github.com/sergiocoding96/hermes-multi-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->

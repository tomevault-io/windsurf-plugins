---
trigger: always_on
description: This supplements `CLAUDE.md` with Codex-specific guidance for this repository.
---

# ECC for Codex CLI

This supplements `CLAUDE.md` with Codex-specific guidance for this repository.

## Project Scope

- This repository is the full MediaTree web/backend project.
- Backend code lives under `backend/app/` and uses FastAPI, SQLite, ffmpeg/ffprobe, and scraper integrations.
- Frontend code lives under `frontend/` and uses React 18, TypeScript, Vite, Tailwind, ArtPlayer, and Capacitor.
- Media under nested `sp` folders is treated as folder-level specials: keep it out of normal listings, scraping, continue watching, and player episode queues unless a specials-specific path is being changed.
- Folder pages prefer TMDB Chinese title logos, then English logos, with the text title as fallback. Episode cards prefer episode stills and then shared folder landscape artwork; preserve shared-image cache reuse and responsive 16:9 sizing when changing this flow.
- Browser playback compatibility includes automatic AC3 transcoding; preserve this behavior when touching stream or player capability code.
- Playback pages update the browser tab title with `▶` / `⏸` and the current media title while the user stays on the page; restore the site title only when leaving playback.
- Scraper cache TTLs and Javdatabase request spacing are internal backend policy: do not re-expose them as Settings/environment configuration. Manual scans, rescrapes, and manual apply paths must bypass scraper cache.
- Keep user-facing explanations, plans, summaries, questions, and change reports in Chinese.
- Keep code identifiers, file names, paths, commands, config keys, API routes, function names, class names, and logs in their original English.

## Product Collaboration Rules

- Treat the user as a product manager with no programming background: translate natural-language requests into product goals, user workflows, and acceptance criteria before choosing the technical implementation.
- Do not require the user to provide technical terminology. When a request is vague, first infer the likely product intent from the repository context, state the interpretation in Chinese, and ask only the minimum necessary clarification if implementation would otherwise be risky.
- Before every code change, read the current project guidance (`AGENTS.md`, `CLAUDE.md`, and any task-relevant docs) so implementation stays aligned with product constraints, release policy, and local conventions.
- When a change is committed, keep the normative docs updated in the same commit whenever the change affects product behavior, architecture, workflows, release/update policy, or future agent instructions.

## Model Recommendations

| Task Type | Recommended Model |
|-----------|------------------|
| Routine coding, tests, formatting | GPT 5.5 |
| Complex features, architecture | GPT 5.5 |
| Debugging, refactoring | GPT 5.5 |
| Security review | GPT 5.5 |

## Skills Discovery

Project skills are available from `.agents/skills/`. Each installed skill contains:

- `SKILL.md` - detailed instructions and workflow
- `agents/openai.yaml` - Codex interface metadata, when provided by ECC

Installed skills:

- `tdd-workflow` - test-driven development with 80%+ coverage expectations
- `security-review` - security checklist and threat review
- `coding-standards` - universal coding standards
- `frontend-patterns` - React/Next.js/frontend patterns
- `frontend-slides` - viewport-safe HTML presentations and PPTX-to-web conversion
- `article-writing` - long-form writing from notes and voice references
- `content-engine` - platform-native social content and repurposing
- `market-research` - source-attributed market and competitor research
- `investor-materials` - decks, memos, models, and one-pagers
- `investor-outreach` - personalized investor outreach and follow-ups
- `backend-patterns` - API design, database, caching
- `e2e-testing` - Playwright E2E tests
- `eval-harness` - eval-driven development
- `strategic-compact` - context management
- `api-design` - REST API design patterns
- `verification-loop` - build, test, lint, typecheck, security
- `deep-research` - multi-source research
- `exa-search` - neural search via Exa MCP
- `x-api` - X/Twitter API integration
- `crosspost` - multi-platform content distribution
- `fal-ai-media` - AI image/video/audio generation via fal.ai
- `dmux-workflows` - multi-agent orchestration

`claude-api` was listed in the upstream ECC inventory but is not present in the local user-level ECC install used to initialize this repo. A placeholder is kept at `.agents/skills/claude-api/` so future installs can fill it without ambiguity.

## MCP Servers

Treat project-local `.codex/config.toml` as the Codex baseline for this repo. The baseline enables multi-agent support and declares the standard ECC MCP server entries used by this project:

- GitHub
- Context7
- Exa
- Memory
- Playwright
- Sequential Thinking
- Supabase

The canonical Context7 section name is `[mcp_servers.context7]`. The launcher package remains `@upstash/context7-mcp`; only the TOML section name is normalized for consistency with `codex mcp list` and the ECC reference config.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZASENJC/mediatree](https://github.com/ZASENJC/mediatree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

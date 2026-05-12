---
trigger: always_on
description: Penny is a local-first AI agent that communicates via Signal, Discord, or a Firefox browser extension. Users send messages, Penny searches the web through the browser extension, reasons using a local LLM (Ollama by default, accessed via the OpenAI Python SDK against any OpenAI-compatible endpoint), and replies in a casual, relaxed style. It runs in Docker with host networking.
---

# CLAUDE.md — Penny Project

## What Is Penny

Penny is a local-first AI agent that communicates via Signal, Discord, or a Firefox browser extension. Users send messages, Penny searches the web through the browser extension, reasons using a local LLM (Ollama by default, accessed via the OpenAI Python SDK against any OpenAI-compatible endpoint), and replies in a casual, relaxed style. It runs in Docker with host networking.

Penny is single-user — a personal assistant deployed locally for one person. Multiple devices (Signal phone, browser instances) connect as different devices of the same user, sharing a single conversation history.

Penny also has an autonomous development team (`penny-team/`) — Claude CLI agents that process GitHub Issues on a schedule, handling requirements, architecture, and implementation.

## Environment Notes

- **Logs**: Runtime logs are written to `data/penny/logs/penny.log`; agent logs are in `data/penny-team/logs/` (not docker compose logs)

## Git Workflow

Branch protection is enabled on `main`. All changes must go through pull requests.

- **Never push directly to `main`** — always create a feature branch
- Create a descriptive branch name (e.g., `add-codeowners-filtering`, `fix-scheduler-bug`)
- Commit changes to the branch, then push and create a PR
- **Use `make token` for GitHub operations** (host only): `GH_TOKEN=$(make token) gh pr create ...`
  - This generates a GitHub App installation token for authenticated `gh` CLI access
  - Agent containers already have `GH_TOKEN` set by the orchestrator — just use `gh` directly
- The user will review and merge the PR

## Documentation Maintenance

**IMPORTANT**: Always update CLAUDE.md and README.md after making significant changes to the codebase. This includes:
- New features or modules
- Architecture changes
- Configuration changes
- API changes
- Directory structure changes

Each sub-project has its own CLAUDE.md — update the relevant one(s).

## Directory Structure

```
penny/                          — Penny chat agent (Signal/Discord)
  penny/                        — Python package
  Dockerfile
  pyproject.toml
  CLAUDE.md                     — Penny-specific context
penny-team/                     — Autonomous dev team (Claude CLI agents)
  penny_team/                   — Python package
  scripts/
    entrypoint.sh               — Docker entrypoint
  Dockerfile
  pyproject.toml
  CLAUDE.md                     — Penny-team-specific context
github_api/                     — Shared GitHub API client (GraphQL + REST)
  api.py                        — GitHubAPI class (typed Pydantic return values)
  auth.py                       — GitHubAuth (App JWT token generation)
similarity/                     — Shared similarity primitives (penny + penny-team)
  embeddings.py                 — Pure math: cosine similarity, TCR, serialization
  dedup.py                      — Dedup strategies (TCR + embedding)
browser/                        — Firefox browser extension
  src/                          — TypeScript source
    protocol.ts                 — Typed WebSocket + runtime messaging protocol
    background/                 — WebSocket owner, tool dispatch, tab tracking
    sidebar/                    — Chat UI, page context toggle
    content/                    — Defuddle-based page extraction (esbuild bundled)
  sidebar/                      — Sidebar HTML + CSS
  icons/                        — Extension icons (rendered from SVG)
  manifest.json                 — WebExtensions manifest
  tsconfig.json                 — Strict TypeScript config
  build-content.mjs             — esbuild wrapper for content script
  package.json                  — Dependencies: defuddle, fontawesome, esbuild, web-ext
Makefile                        — Dev commands (make up, make check, make prod)
docker-compose.yml              — signal-api + penny + team services
docker-compose.override.yml     — Dev source volume overrides
scripts/
  watcher/                      — Auto-deploy service
.github/
  workflows/
    check.yml                   — CI: runs make check on push/PR to main
  CODEOWNERS                    — Trusted maintainers (used by penny-team filtering)
docs/                           — Design documents and review guides
  pr-review-guide.md            — Canonical PR review checklist (used by /quality skill)
  browser-extension-architecture.md — Browser extension architecture & design
  channel-manager-plan.md       — Multi-channel implementation plan
  browser-tools-plan.md         — Browser tools implementation plan
  agent-memory-patterns.md      — Patterns for agent memory recall and dedup
  benchmarking-embedding-models.md — Embedding model benchmark results
  benchmarking-qwen35-vs-gpt-oss.md — qwen3.5 vs gpt-oss benchmark comparison
data/                           — Runtime data (gitignored)
  penny/                        — Penny runtime data
    penny.db                    — Production database

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaredlockhart/penny](https://github.com/jaredlockhart/penny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

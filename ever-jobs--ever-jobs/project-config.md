---
trigger: always_on
description: > This file is the single source of truth for agents (Claude Code, OpenAI Codex, Cursor, etc.)
---

# AGENTS.md — Ever Jobs (Authoritative Rules for AI Agents)

> This file is the single source of truth for agents (Claude Code, OpenAI Codex, Cursor, etc.)
> working on the **Ever Jobs** monorepo. Cross-check every spec, doc, and code file you create
> or edit against the rules in this document.

---

## 0. North Star

Ever Jobs is a **modular, plugin-driven, TypeScript-only NestJS monorepo** for multi-source
job scraping & analysis. It must be:

1. **Modular** — plugins/adaptors for almost every feature (sources, ATS, AI, exports, UI).
2. **Replaceable** — every plugin can be enabled / disabled / swapped at runtime.
3. **Performant** — extreme runtime performance: streaming, pooling, caching, parallelism.
4. **Documented for AI** — `docs/` and `.specify/` answer every reasonable agent question.

---

## 1. Source-of-Truth Documents

| File / Folder                | Role                                                                |
| ---------------------------- | ------------------------------------------------------------------- |
| `AGENTS.md` (this file)      | Authoritative rules for all coding agents.                          |
| `CLAUDE.md`                  | Claude-specific operating notes (loads `AGENTS.md` by reference).   |
| `docs/`                      | All long-form specs, plans, ADRs, runbooks, glossaries.             |
| `docs/index.md`              | Full index of every doc — keep in sync.                             |
| `docs/log.md`                | Append-only changelog of doc/spec changes (newest at top).          |
| `docs/questions.md`          | Open questions; agents add options & a default; humans review.      |
| `.specify/`                  | GitHub Spec Kit directory: `memory/`, `specs/`, `templates/`.       |
| `tool_manifest.json`         | Machine-readable description of API endpoints & sources.            |
| `README.md`                  | User-facing overview; do not duplicate spec content here.           |

---

## 2. Hard Rules (do not violate)

1. **TypeScript only.** No JavaScript runtime sources, no Python, no shell-pipeline logic
   beyond build/devops scripts. Tests, CLI commands, plugins → all `.ts`.
2. **Spec-Kit-first.** No coding without a written spec under `docs/specs/` (or `.specify/specs/`)
   that includes: problem statement, scope, non-goals, contracts, test plan.
3. **Plugin-by-default.** New features = new package under `packages/plugin/` or
   `packages/plugins/<plugin-id>/`. The core (`apps/api`, `apps/cli`) only orchestrates.
4. **Replaceable contracts.** Every plugin implements an interface from `@ever-jobs/models`
   or `@ever-jobs/plugin`; no plugin imports another plugin directly.
5. **Latest deps.** Always prefer the latest stable versions of dependencies; record
   non-trivial bumps in `docs/log.md`.
6. **Reuse existing libs** when popular & well-maintained (Cheerio, Playwright, Axios,
   Cache Manager, BullMQ, Zod, etc.) — do not reinvent.
7. **Tests required.** Unit tests live next to plugin source (self-contained packages);
   e2e tests collocate under `apps/api/__tests__/e2e/` or `apps/<app>/__tests__/e2e/`.
8. **Performance.** Default to: streaming responses, async iterators, connection pools,
   bounded concurrency (`p-limit`/`Promise.allSettled`), Redis cache, structured indexes.
9. **No deletion.** Do not delete user-authored files; *move* or *improve* in place.
   Mark deprecated code with `@deprecated` and a removal target.
10. **Security.** All HTTP I/O goes through `@ever-jobs/common` HTTP client (UA rotation,
    timeouts, retries, redacted logging). Never log secrets.
11. **Be exhaustive.** Don't summarize — write full specs, full task lists, full test plans.

---

## 3. Repository Layout (canonical)

```
ever-jobs/
├── AGENTS.md                # this file
├── CLAUDE.md                # Claude operating notes
├── README.md
├── package.json             # npm workspaces (apps/* + packages/*)
├── tsconfig.base.json       # path aliases for every package
├── nx.json                  # nx task graph
├── .specify/                # GitHub Spec Kit
│   ├── memory/              #   constitution, principles
│   ├── specs/               #   per-feature specs (ID-prefixed)
│   └── templates/           #   spec/plan/tasks templates
├── docs/
│   ├── index.md             # auto-maintained doc index
│   ├── log.md               # append-only changelog of doc edits
│   ├── questions.md         # open questions for the human owner
│   ├── adr/                 # architectural decision records
│   ├── plans/               # implementation plans (one per feature)
│   ├── specs/               # functional specs (mirror or alias of .specify/specs/)
│   ├── runbooks/            # operational guides
│   └── *.md                 # legacy / reference docs (keep, never delete)
├── apps/
│   ├── api/                 # NestJS HTTP/GraphQL/MCP server
│   ├── cli/                 # nest-commander based CLI
│   ├── mcp/                 # standalone MCP server (AI-agent integration)
│   └── web/                 # (future) frontend dashboard plugin host
├── packages/
│   ├── plugin/              # core plugin infra (registry, decorator, discovery)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ever-jobs/ever-jobs](https://github.com/ever-jobs/ever-jobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

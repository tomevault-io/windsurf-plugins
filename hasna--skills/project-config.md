---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
bun install                       # Install dependencies
bun run build                     # Build CLI, MCP server, library, and type declarations
bun run dev                       # Run CLI in development mode (bun run src/cli/index.tsx)
bun test                          # Run all tests (Bun native test runner)
bun test src/lib/registry.test.ts # Run a single test file
bun run typecheck                 # Type-check without emitting (tsc --noEmit)
bun run dashboard:build           # Build Next.js dashboard (cd dashboard && bun install && bun run build)
bun run dashboard:dev             # Next.js dev server with HMR (port 3505, proxies /api to :3579)
bun run server                    # Start HTTP server (port 3579, serves dashboard + API)
bun run server:dev                # Start server with --watch
```

## Architecture

Monorepo containing 202 AI agent skills plus a framework for discovering, installing, and managing them. Four interfaces share a common set of core modules:

```
src/
├── cli/
│   ├── index.tsx                 # Commander.js CLI + Ink TUI entry
│   ├── cli.test.ts
│   └── components/
│       └── App.tsx               # Interactive TUI (React/Ink)
├── mcp/
│   ├── index.ts                  # MCP server (stdio transport)
│   └── mcp.test.ts
├── server/
│   ├── serve.ts                  # Bun.serve HTTP server + REST API
│   └── serve.test.ts
├── lib/
│   ├── registry.ts               # SKILLS array (202 entries) + CATEGORIES (17)
│   ├── installer.ts              # Install/remove to .skills/ and agent dirs
│   ├── skillinfo.ts              # Docs, requirements, env var extraction, run
│   ├── utils.ts                  # normalizeSkillName()
│   ├── registry.test.ts
│   ├── installer.test.ts
│   ├── skillinfo.test.ts
│   ├── skillinfo-run.test.ts
│   ├── utils.test.ts
│   └── validation.test.ts        # Structural validation: all 202 skills
├── index.ts                      # Library re-exports
└── index.test.ts

dashboard/                        # Vite + React 19 + Tailwind v4 + shadcn/ui
├── src/components/
│   ├── skills-table.tsx          # TanStack Table for skills
│   ├── skill-detail-dialog.tsx
│   ├── stats-cards.tsx
│   ├── theme-provider.tsx        # Dark/light/system (oklch tokens)
│   ├── theme-toggle.tsx
│   └── ui/                       # shadcn/ui primitives
├── package.json
├── vite.config.ts
└── tsconfig.json

skills/                           # 202 self-contained skill directories
├── _common/                      # Shared utilities for skills
├── skill-image/
├── skill-deep-research/
├── ...
└── tsconfig.base.json
```

### Interfaces

**CLI (`src/cli/index.tsx`)** -- Commander.js with Ink (React for terminal). Running `skills` with no args launches an interactive TUI. Subcommands: `install`, `list`, `search`, `info`, `docs`, `requires`, `run`, `remove`, `update`, `categories`, `init`, `mcp`, `serve`, `self-update`.

**MCP Server (`src/mcp/index.ts`)** -- Model Context Protocol server over stdio. 9 tools (`list_skills`, `search_skills`, `get_skill_info`, `get_skill_docs`, `install_skill`, `remove_skill`, `list_categories`, `get_requirements`, `run_skill`) and 2 resources (`skills://registry`, `skills://{name}`).

**HTTP Server** — not shipped in OSS. The SaaS dashboard and `/api/v1/*` endpoints live in the private `platform-skills` repo.

**Library (`src/index.ts`)** -- npm package `@hasna/skills` re-exporting registry, installer, and skillinfo modules.

### Core Modules

**`src/lib/registry.ts`** -- The `SKILLS` array (202 entries) with `SkillMeta` interface (name, displayName, description, category, tags) and `CATEGORIES` tuple (17 categories). Functions: `getSkill()`, `getSkillsByCategory()`, `searchSkills()`.

**`src/lib/installer.ts`** -- Copies skill source into `.skills/` in the user's project. Updates `.skills/index.ts` on every install/remove. Also supports agent-specific installs (copies SKILL.md to `~/.claude/skills/`, `~/.codex/skills/`, or `~/.gemini/skills/`). Functions: `installSkill()`, `installSkills()`, `removeSkill()`, `getInstalledSkills()`, `installSkillForAgent()`, `removeSkillForAgent()`, `resolveAgents()`.

**`src/lib/skillinfo.ts`** -- Reads docs (priority: SKILL.md > README.md > CLAUDE.md), extracts env vars and system deps via regex patterns, reads CLI command from package.json bin field, generates SKILL.md from metadata if missing, can execute skills via `runSkill()` (auto-installs deps, spawns via Bun). Functions: `getSkillDocs()`, `getSkillBestDoc()`, `getSkillRequirements()`, `runSkill()`, `generateEnvExample()`, `generateSkillMd()`.

**`src/lib/utils.ts`** -- `normalizeSkillName()` which prefixes `skill-` if missing.

## Key Patterns

### Skill Name Normalization

Registry uses bare names (`image`), filesystem uses prefixed names (`skill-image`). `normalizeSkillName()` in `src/lib/utils.ts` handles conversion. This applies everywhere: installer, skillinfo, server, MCP.

### Installation Modes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hasna/skills](https://github.com/hasna/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

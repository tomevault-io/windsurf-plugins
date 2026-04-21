---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm build          # Build with obuild
pnpm dev:prepare    # Stub for development (obuild --stub)
pnpm typecheck      # TypeScript check (tsc --noEmit)
pnpm lint           # ESLint (@antfu/eslint-config)
pnpm lint:fix       # ESLint with auto-fix
pnpm test           # Run vitest in watch mode
pnpm test:run       # Run vitest once (CI-style)
pnpm test -- test/unit/cache.test.ts     # Single test file
pnpm test -- --project unit              # Unit tests only
pnpm test -- --project e2e               # E2E tests only
```

### CLI Commands

```bash
skilld                # Interactive menu
skilld add vue nuxt   # Add skills for packages
skilld update         # Update all outdated skills
skilld update vue     # Update specific package
skilld remove         # Remove installed skills
skilld list           # List installed skills (one per line)
skilld list --json    # List as JSON
skilld info           # Show config, agents, features, per-package detail
skilld config         # Change settings
skilld install        # Restore references from lockfile
skilld prepare        # Hook for package.json "prepare" (restore refs, sync shipped, report outdated)
skilld uninstall      # Remove skilld data
skilld search "query" # Search indexed docs
skilld search "query" -p nuxt  # Search filtered by package
skilld cache --clean     # Clean expired LLM cache entries
skilld cache --stats     # Show cache disk usage breakdown
skilld add owner/repo    # Add pre-authored skills from git repo
skilld eject vue                    # Eject skill (portable, no symlinks)
skilld eject vue --name vue         # Eject with custom skill dir name
skilld eject vue --out ./dir/       # Eject to custom path
skilld eject vue --from 2025-07-01  # Only releases/issues since date
skilld author                       # Generate skill for npm publishing (monorepo-aware)
skilld author -m haiku              # Author with specific LLM model
skilld author -o ./custom/          # Author to custom output directory
```

## Architecture

CLI tool that generates AI agent skills from NPM package documentation. Requires Node >= 22.6.0. Flow: `package name → resolve docs → cache references → generate SKILL.md → install to agent dirs`.

**Key directories:**
- `~/.skilld/` - Global cache: `references/<pkg>@<version>/`, `llm-cache/`, `config.yaml`
- `.claude/skills/<pkg>/SKILL.md` - Generated skill files (project-level)
- `src/commands/` - CLI subcommands routed via citty `subCommands` in cli.ts
- `src/agent/` - Agent registry, detection, LLM spawning, skill generation
  - `clis/` - LLM CLI integrations (claude, codex, gemini)
  - `prompts/` - Skill generation prompt templates; `optional/` for toggleable sections (api-changes, best-practices, types, etc.)
  - `targets/` - Per-agent target definitions
- `src/sources/` - Doc fetching (npm registry, llms.txt, GitHub via ungh.cc)
- `src/cache/` - Reference caching with symlinks to `~/.skilld/references/`
- `src/retriv/` - Vector search with sqlite-vec + @huggingface/transformers embeddings
- `src/core/` - Config (custom YAML parser), skills iteration, formatting, lockfile

**Doc resolution cascade (src/commands/sync.ts):**
1. Package ships `skills/` directory → symlink directly (skills-npm convention)
2. Git-hosted versioned docs → fetch from GitHub tags via ungh.cc
3. Registry `crawlUrl` → crawl specific URL pattern via `@mdream/crawl` (e.g. motion-v)
4. `llms.txt` at package homepage → parse and download linked .md files
5. Website crawl → crawl `docsUrl/**` via sitemap when no docs found above
6. GitHub README via ungh proxy → fallback

Resolution tracked via `ResolveAttempt[]` array for debugging failures. Blog release posts (curated in `src/sources/blog-presets.ts`) supplement docs for major version announcements.

**Git skills (`src/sources/git-skills.ts`, `src/commands/sync-git.ts`):**
Installs pre-authored skills from git repos. Accepts `owner/repo`, full URLs, SSH, or local paths. Clones/pulls into `~/.skilld/git-skills/`, copies `skills/` directory contents. Part of skills-npm ecosystem compatibility.

**LLM integration (NO AI SDK):**
Spawns CLI processes directly (`claude`, `gemini`) with `--add-dir` for references. Custom stream-json parsing for progress. Results cached at `~/.skilld/llm-cache/<sha256>.json` with 7-day TTL.

**Agent detection (`src/agent/detect.ts`):**
Checks env vars (`CLAUDE_CODE`, `CURSOR_SESSION`) and project dirs (`.claude/`, `.cursor/`) to auto-detect target. Registry in `src/agent/registry.ts` defines per-agent skill dirs and detection. Supports 11 agents: claude-code, cursor, windsurf, cline, codex, github-copilot, gemini-cli, goose, amp, opencode, roo.

**Import scanning (`src/agent/detect-imports.ts`):**
AST-based import detection via oxc-parser. `FILE_PATTERN_MAP` in `src/agent/types.ts` maps ~98 packages to file glob patterns (e.g., `vue` → `*.vue`) for efficient scanning. `detect-presets.ts` handles framework-specific package discovery (e.g., Nuxt modules).

**Cache structure:**
```
~/.skilld/references/<pkg>@<version>/
  docs/          # Fetched external docs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skilld-dev/skilld](https://github.com/skilld-dev/skilld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

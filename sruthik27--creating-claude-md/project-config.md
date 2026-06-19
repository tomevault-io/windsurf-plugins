---
trigger: always_on
description: Use when the user wants to create a new CLAUDE.md from scratch for a repository, bootstrap project memory for a new repo, set up a repo for Claude Code for the first time, or completely rewrite an existing CLAUDE.md deemed unsalvageable. Produces a lean, high-signal file under 80 lines (120 hard cap) by scanning the repo for mechanical content (stack, commands, structure) and interviewing the user for judgment content (WHY, gotchas, out-of-scope, approval gates, testing philosophy, external docs
---


# Creating a CLAUDE.md From Scratch

Generate a lean, high-signal CLAUDE.md for a repository by combining an autonomous repo scan with a focused 6-question user interview.

**Core principle:** Every line must earn its place. If removing a line would not cause Claude to make a specific mistake, it does not belong in the file.

## When to Use vs. Not Use

**Use when:**
- Repo has no CLAUDE.md and user wants one.
- Repo has a CLAUDE.md that the user considers unsalvageable and wants rewritten from scratch.
- User says "set this repo up for Claude Code", "bootstrap project memory", "write me a CLAUDE.md".

**Do NOT use when:**
- User wants to audit, improve, or incrementally fix an existing CLAUDE.md. Route to `claude-md-improver` instead.
- User wants to add a single rule / gotcha to an existing file. Edit directly.
- Repo is a scratch directory with no meaningful content yet.

## The Iron Rule

**Target ≤ 80 lines. Hard cap 120.** This is a forcing function. Bloat triggers the harness to uniformly down-weight all instructions in the file (not just the tail). If the draft exceeds 80 lines, split oversized sections into `agent_docs/*.md` before writing anything to disk.

## 4-Phase Workflow

Follow in order. Do not skip phases.

### Phase 1: Scan the Repo (Autonomous)

Gather mechanical content the repo can tell you:

- **Manifests** — read whichever exist: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`, `Makefile`. Extract: language, framework, versions, scripts.
- **Structure** — `ls` top-level + one level deep. Detect monorepo via `workspaces` in `package.json`, `pnpm-workspace.yaml`, `lerna.json`, `turbo.json`, `nx.json`, or `packages/` / `apps/` directories.
- **Enforced conventions** — read `.editorconfig`, `.eslintrc*`, `biome.json`, `.prettierrc*`, `ruff.toml`, `pyproject.toml` (`[tool.ruff]` / `[tool.black]`), `rustfmt.toml`, `.golangci.yml`. **Record these — the corresponding rules are BANNED from CLAUDE.md** (linter does the job).
- **Existing docs** — `README.md`, any `agent_docs/`, `.claude/rules/`, subdirectory `CLAUDE.md` files. Do not overwrite auxiliary files that already exist without asking.
- **Git** — `git log --oneline -20` to detect commit conventions (Conventional Commits, prefix patterns).
- **Boundary check** — look for existing `CLAUDE.md`, `AGENTS.md`, `CLAUDE.local.md`. If `CLAUDE.md` exists with meaningful content (> 20 lines, not just a stub), STOP and ask user: `[rewrite / run claude-md-improver / cancel]`. Proceed only if they confirm rewrite.
- **Auto-memory** — check `~/.claude/projects/<path-hash>/memory/` if it exists. Note what Claude has already auto-learned so you do not duplicate it.
- **External-doc MCPs** — check which MCPs are configured in this session for external docs. Common names include `confluence-dc`, `confluence`, `notion`, `atlassian`. Record the exact MCP name (e.g., `confluence-dc`) so Phase 3 can reference it by the same name in the generated `## External Docs` section. If none are configured but the user references external docs in Q6, record that as plain URLs without MCP instructions.

### Phase 2: Interview the User

Ask all questions in a **single numbered list**. User answers all together. Do not ask one at a time.

**Fixed 6 (always ask):**

1. **WHY** — In one sentence, what does this project do and who uses it?
2. **Gotchas** — What has tripped up new contributors or Claude in this repo? Non-obvious traps, modules with weird behavior, required setup steps not in the README.
3. **Out of scope** — Any files, directories, or areas Claude should NEVER modify autonomously? (auto-generated files, migrations, infra, other teams' code)
4. **Approval gates** — Any actions that require your explicit approval before Claude runs them? (destructive git, DB schema changes, deploys, production-adjacent changes)
5. **Testing philosophy** — How do you verify changes work? Is there any check that must always pass before shipping?
6. **External docs** — Are there Confluence pages (or Notion / wiki / Google Docs) that document parts of this repo? For each, paste: (a) the URL, (b) a one-line description of what it covers, (c) policy — `read` (reference only, never modify) or `update` (keep in sync when related code changes). Leave blank if none.

**Adaptive follow-ups (add up to 3, only when the scan surfaced ambiguity):**

- Monorepo detected → "I see N packages: X, Y, Z. Which ones does Claude frequently confuse or work across incorrectly?"
- `migrations/` directory found → "Any rules around creating or running migrations?"
- Multiple lockfiles (`package-lock.json` + `yarn.lock` + `pnpm-lock.yaml`) → "Which package manager is authoritative?"
- CI config present but no local check script → "Any CI checks that must pass locally before pushing?"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sruthik27/creating-claude-md](https://github.com/sruthik27/creating-claude-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

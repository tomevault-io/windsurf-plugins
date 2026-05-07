---
trigger: always_on
description: This file gives repository-specific guidance to AI coding agents working on `better-web-ui`.
---

# AGENTS.md

This file gives repository-specific guidance to AI coding agents working on `better-web-ui`.

## Repository purpose

`better-web-ui` is a public Agent Skills package focused on high-quality frontend design work.

It ships:

- a canonical skill package in `skills/`
- shared design doctrine in `skills/frontend-design/reference/`
- generated compatibility wrapper trees for agent-specific layouts
- human-facing docs in `README.md`
- attribution and license context in `NOTICE.md` and `LICENSE`

This is **not** a generic skills starter and **not** a Vercel deployment repo. Keep docs, examples, and contribution guidance specific to this project.

## Source of truth

The canonical source of truth is always:

```text
skills/{skill-name}/
```

Edit canonical skills there.

If a wrapper tree and `skills/` ever disagree, `skills/` is the source of truth.

Wrapper trees such as `.github/skills/...` or `.claude/skills/...` are generated compatibility shims and should not contain original doctrine.

## Repository layout

```text
skills/
  {skill-name}/
    SKILL.md
    reference/        # optional shared or skill-specific docs
    references/       # also acceptable if used intentionally
    scripts/          # optional deterministic helpers
    assets/           # optional templates or static assets

.agents/skills/
.github/skills/
.claude/skills/
.codex/skills/
.cursor/skills/
.opencode/skills/
.pi/skills/
  README.md           # generated wrapper-root guidance
  {skill-name}/
    SKILL.md          # generated wrapper pointing back to skills/{skill-name}/SKILL.md

README.md
CONTRIBUTING.md
DEVELOPMENT.md
CHANGELOG.md
AGENTS.md
CLAUDE.md             # should remain a pointer to AGENTS.md
NOTICE.md
LICENSE
```

## Working rules

### Tooling baseline

- Use Node `24.14.1` for local maintainer work.
- Treat `>=24.14.1 <25` as the supported engine range for repository tooling and CI.
- Run `npm install` before using repository scripts.
- Use `npm run lint` for repository scripts, `npm run generate:wrappers` for compatibility trees, `npm run check:wrapper-drift` for generated wrapper diff checks, and `npm run validate` for canonical skill, doc, and wrapper checks.
- The repository generates all supported wrapper roots up front; it does **not** decide which root a host installs into.
- Host/editor detection during `npx skills add ...` is owned by the external `skills` CLI. Treat upstream agent selection and path routing as installer behavior rather than as a wrapper-generation bug in this repository.
- For current maintainer docs and examples, default public install guidance to the explicit supported target set `--agent codex --agent cursor --agent github-copilot --agent opencode`. Upstream intentionally routes those agents through the shared `.agents/skills/` project harness even though this repository also publishes `.github/skills/`, `.codex/skills/`, `.cursor/skills/`, and `.opencode/skills/` compatibility wrappers.
- The upstream `skills` CLI treats every agent with project path `.agents/skills/` as part of its locked Universal group during interactive installs. This repository cannot narrow that Universal group through skill metadata; use explicit `--agent` examples instead.
- Do not recommend `npx skills add ... --all` in public docs unless you explicitly mean all skills to all agents. Prefer explicit supported `--agent` examples instead.

### 1. Edit canonical skills first

- Make content changes only in `skills/`
- Regenerate wrappers after adding, removing, renaming, or changing frontmatter for any skill
- Do not hand-maintain divergent logic across wrapper trees

### 2. Keep skill metadata portable

For each `SKILL.md`:

- `name` must match the directory name
- `description` must explain both **what the skill does** and **when to use it**
- prefer imperative descriptions that explicitly tell the agent when to act, usually with `Use when...`
- lead with user intent and outcomes before internal mechanics or implementation details
- include trigger language broad enough to catch realistic prompts, and add boundaries when adjacent skills overlap
- keep names lowercase and hyphenated
- prefer concise, trigger-friendly descriptions that stay comfortably under the 1024-character spec limit so startup metadata stays lean

If a skill needs an argument hint, store it under `metadata.argument-hint` rather than as a top-level frontmatter field so the skill remains valid against the current Agent Skills validator.

### 3. Prefer progressive disclosure

- Keep `SKILL.md` focused on workflow
- Move reusable doctrine into reference files when a concept appears in 3+ skills
- Prefer pointing multiple skills to `skills/frontend-design/reference/` over duplicating long guidance blocks

### 4. Keep wrappers thin

Each wrapper should:

- copy the canonical frontmatter
- state that it is a compatibility wrapper
- link to `../../../skills/{skill-name}/SKILL.md`

Do not duplicate the full skill body into wrapper trees.

Do not add host-detection logic or editor-specific installation rules to this repository. Those belong in the upstream installer, while this repository only needs to keep the generated wrapper trees accurate and in sync.

### 5. Preserve project identity


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aladicf/better-web-ui](https://github.com/aladicf/better-web-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

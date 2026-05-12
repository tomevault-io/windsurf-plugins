---
trigger: always_on
description: Quick reference for NVCF (NVIDIA Cloud Functions) in this repository.
---

# AGENTS.md - Guide for AI Coding Agents

Quick reference for NVCF (NVIDIA Cloud Functions) in this repository.

## Repo Layout

This repo is an umbrella layout: upstream services appear as ordinary directories (synthetic imports), arranged under `src/`, `deploy/`, `infra/`, and `migrations/` according to `imports.yaml`. Goal: over time, land and maintain code here natively; synthetic imports are a bridge while sources still live in separate GitLab projects. Tooling lives under `tools/` and `tests/`.

Use `python3`, not `python`, when Python is needed. Use the nearest nested `AGENTS.md` for subtree-specific guidance.

Useful pointers:
- `tools/AGENTS.md` for repo tooling
- `.cursor/skills/add-synthetic-import/SKILL.md` for synthetic imports
- `.cursor/skills/documentation-style/SKILL.md` for docs style
- `.cursor/skills/` for root dev-skill symlink fanout
- `ai-tooling/user/skills/` and `ai-tooling/dev/skills/` for public skills
- `nvidia-internal/user/skills/` and `nvidia-internal/dev/skills/` for private skills

If a referenced skill is outdated, update it before finishing.

## Writing AGENTS.md Files

Every subtree that an agent may work in should have its own `AGENTS.md` with build commands, test commands, code style, and any subtree-specific conventions. Keep each file under 400 lines; split into separate docs or skills when it grows past that.

`AGENTS.md` is the source of truth for agent guidance. Cursor and Codex read `AGENTS.md` directly. Claude Code reads `CLAUDE.md`, so every directory that has an `AGENTS.md` also has a sibling `CLAUDE.md` that is a regular file containing the single line `@AGENTS.md`. That import line tells Claude Code to load the adjacent `AGENTS.md`, so all three tools end up on the same content. When creating a new `AGENTS.md`, create the companion `CLAUDE.md` in the same commit: `printf '@AGENTS.md\n' > CLAUDE.md`. Do not use a symlink, and never put unique content in `CLAUDE.md`.

## Skills

Skills are reusable, on-demand agent instructions for specific workflows. They follow the [Agent Skills specification](https://agentskills.io/specification) and are compatible with the [Vercel Skills CLI](https://github.com/vercel-labs/skills). Skills are invoked when relevant, not auto-applied (auto-applied guidance belongs in rules, not skills).

### Skill structure

Each skill is a directory named to match its `name` frontmatter field, containing at minimum a `SKILL.md`. Names must be lowercase with hyphens only, no leading/trailing/consecutive hyphens.

```
skill-name/
    SKILL.md              # Required (under 500 lines)
    README.md             # Optional: overview and usage
    examples.md           # Optional: detailed examples
    references/           # Optional: reference docs
    scripts/              # Optional: helper scripts
    assets/               # Optional: images, diagrams
```

### SKILL.md frontmatter

```yaml
---
name: skill-name
description: >-
  What the skill does and when to use it.
  Include trigger keywords for discoverability.
version: "1.0.0"
tags:
  - nvcf
  - relevant-tag
tools:
  - Shell
  - Read
---
```

The `description` must say both what the skill does (actions it enables) and when to use it (trigger phrases, keywords). This is how agents decide whether to invoke the skill.

### Where skills live

Skills are split by visibility and audience:

- `ai-tooling/user/skills/`: public user-facing NVCF skills.
- `ai-tooling/dev/skills/`: public developer workflow skills.
- `nvidia-internal/user/skills/`: private user-facing NVCF skills.
- `nvidia-internal/dev/skills/`: private developer, release-engineering, and monorepo-maintenance skills.
- `.cursor/skills/`: root dev-skill fanout only. Each entry is a symlink to a public dev source under `ai-tooling/dev/skills/` or a private dev source under `nvidia-internal/dev/skills/`.

Cross-tool symlinks make root dev skills available to all agents:
- `.cursor/skills/<name>` -> symlink to the dev skill source directory.
- `.codex/skills/<name>` -> symlink to the same dev skill source directory.
- `.claude/skills/<name>` -> symlink to the same dev skill source directory.

When adding a skill:
1. Decide visibility (`ai-tooling` public or `nvidia-internal` private) and audience (`user/skills` or `dev/skills`).
2. Create the `SKILL.md` with valid frontmatter.
3. For root-wide dev skills, create matching `.cursor/skills/<name>`, `.codex/skills/<name>`, and `.claude/skills/<name>` symlinks to the same source directory.
4. Update the relevant public or private skills table.

### Public skills

| Skill | Location | Purpose |
|-------|----------|---------|
| `documentation-style` | `ai-tooling/dev/skills/` | NVCF documentation conventions (no bold, no emojis, no em-dash) |
| `nvcf-explore-stack` | `ai-tooling/dev/skills/` | Navigate the self-hosted stack topology and dependency graph |
| `nvcf-self-managed-cli` | `ai-tooling/user/skills/` | Install, operate, and manage self-managed NVCF through `nvcf-cli` |
| `nvcf-self-managed-installation` | `ai-tooling/user/skills/` | Install and deploy the self-managed NVCF stack |

Private skill inventory lives in `nvidia-internal/AGENTS.md`.

## Commit Messages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/nvcf](https://github.com/NVIDIA/nvcf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->

---
trigger: always_on
description: > **Lost? Start at [`START-HERE.md`](START-HERE.md)** — status at a glance + which doc is canonical vs frozen vs archived.
---

# CLAUDE.md

> **Lost? Start at [`START-HERE.md`](START-HERE.md)** — status at a glance + which doc is canonical vs frozen vs archived.
> **Also read `AGENTS.md`** — it contains shared instructions for all AI agents working in this repo.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A multi-agent orchestration toolkit for Claude Code — 73 OSS-publishable skills in `skills/`. Skills are symlinked to `~/.claude/skills/` for global availability.

The toolkit targets Claude Code as the primary host but the SKILL.md format is platform-agnostic — Claude.ai, Copilot CLI, Codex, and Gemini CLI all consume it. Skills should describe work in terms of capabilities ("read the file", "run the command") rather than Claude-Code-specific tool names where reasonable, so the same skill body works across hosts.

## Install / Sync

Use the `/sync-skills` command to create symlinks from `~/.claude/skills/` back to this repo. This keeps skills always in sync — edits in either location are reflected immediately.

```bash
# Via slash command (recommended)
/sync-skills

# Manual: create category symlinks + flattened discovery symlinks
# See skills/workflows/sync-skills/SKILL.md for details
```

## Skill Anatomy

Every skill follows this structure:

```text
skill-name/
├── SKILL.md              # YAML frontmatter + markdown instructions (≤5,000 words; warn at 500 lines)
└── references/           # On-demand reference files (unlimited size)
```

All SKILL.md files use the frontmatter convention defined in `skills/meta/skill-writer/references/frontmatter-spec.md`. This spec aligns with Anthropic's official Agent Skills standard. Required fields: `name` (kebab-case; `claude-*`/`anthropic-*` prefixes discouraged but allowed as documented exceptions), `version` (semver, top-level), `description` (trigger text, ≤1024 chars, no `<` or `>` in field values). Optional Anthropic fields: `compatibility`, `license`, `allowed-tools` (hyphen canonical; `allowed_tools` accepted as alias), `metadata`. Agent roles also declare `owns`, `composes_with`, `spawned_by`.

## Skill Categories

- **`skills/orchestrator/`** (1) — Entry point. 14-phase build playbook, runtime detection, contract-first coordination. References: phase-guide, team-sizing, circuit-breaker, handoff-protocol.
- **`skills/roles/`** (10) — Implementation agents (backend, frontend, infrastructure, qe, security, performance, observability, docs, db-migration, code-review). Each has a SKILL.md + reference files with validation checklists.
- **`skills/contracts/`** (2) — contract-author (generates contracts from templates) and contract-auditor (verifies implementations match). Templates: OpenAPI, AsyncAPI, Pydantic, TypeScript, JSON Schema.
- **`skills/meta/`** (7) — skill-writer, skill-review, skill-update, skill-explorer, skill-catalog, madness (the front-door router: reads intent, picks the right starting skill — orchestrator / plan-builder / a loop / a role or workflow / skill-explorer — and launches it, confirming before anything expensive; the active counterpart to skill-explorer), model-adaptation (the model-migration reference: what to prune, what now triggers a `reasoning_extraction` refusal, and the long-run/effort scaffolding to add when the underlying Claude model changes — Fable 5 / Mythos 5 today; enforced via skill-review, loop-controller, and orchestrator).
- **`skills/git/`** (4) — Git workflow conventions: git-commit, git-pr, git-pr-feedback, git-post-merge-cleanup.
- **`skills/workflows/`** (36) — plan-builder, plan-intake, living-plan, context-manager, deployment-checklist, dependency-coordinator, project-profiler, wiki-research, interactive-doc, settings-consolidator, sync-skills, ui-brief, claude-design-brief, mermaid-charts, nano-banana, artifact-publish, playwright, render-sanity, design-token-guard, class-extraction-guard, repo-deep-dive, llm-wiki, railway-deploy, use-freellmapi, architecture-rescue, caveman, diagnose-loop, grill-me, find-unknowns, maintain-context, zoom-out, setup-project-skills, work-item-brief, website-walkthrough-video, use-pxpipe, yagni-gate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivy00johns/Skill-Madness](https://github.com/ivy00johns/Skill-Madness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

---
trigger: always_on
description: This file is the development guide for this skills-registry repo itself.
---

# AGENTS.md

This file is the development guide for this skills-registry repo itself.

Use it when this repository is the active project and the user wants to improve the skill system, not just operate it from another project.

## Read Order

1. [README.md](README.md)
2. [skills/shared/manage-agent-skills/SKILL.md](skills/shared/manage-agent-skills/SKILL.md)
3. This file

## Repo Purpose

This template is the source-of-truth catalog and install-management system for AI agent skills across:

- Antigravity
- Claude Code
- Codex
- Gemini CLI

This repo is both:

- a catalog of canonical skill sources
- a small product that generates agent-global and project-local install views from manifests
- a universal shared meta-skill that teaches agents how to operate the system safely

## Template Boundary

This repo is the public, distributable template version of the system. It is not the user's lived-in private registry.

Treat it as:

- the productized minimal starter
- the public distribution surface
- the place where reusable system behavior should remain clean and understandable to new users

Do not let it drift into a copy of the private live registry.

The private sibling repo currently lives at:

- [agent-skills](/Users/yammaku/Documents/Projects/agent-skills)

## Document Roles

Treat the docs as layered on purpose:

- [README.md](README.md): front door, bootstrap instructions, and high-level system model
- [skills/shared/manage-agent-skills/SKILL.md](skills/shared/manage-agent-skills/SKILL.md): cross-project operator manual after bootstrap
- [AGENTS.md](AGENTS.md) and [CLAUDE.md](CLAUDE.md): repo-local development guide for evolving this system itself
- [README.md](README.md) is intentionally the AI-led onboarding contract for fresh install and migration. Do not split that into a separate bootstrap skill.

If a rule is necessary for correct behavior from any project, it belongs in the meta-skill. If a rule is about improving this repo as a product, it belongs here.

The intended cross-project workflow is:

1. an installed agent starts from `manage-agent-skills`
2. if the task is ordinary operation, it stays in the meta-skill workflow
3. if the task is system evolution, the meta-skill should direct the agent here

So this file is not the day-to-day entrypoint. It is the deeper development context for changing the system itself.

## Core Model

Keep these layers separate:

1. Catalog
   Canonical skill sources under `skills/`
2. Agent adapters
   Compatibility files under `agents/` that describe each harness's native paths and install strategy
3. Agent-global install views
   Generated managed install views under `installs/agent-global/<agent>`
4. Project install views
   Generated symlink views under the project-local paths declared by each agent adapter

Keep these concepts separate too:

- `shared` means cross-agent compatible source category
- `agent-specific` means source category for one agent
- `agent-global` means installed for one agent across projects on one machine
- `project` means installed only for one project

Never treat `shared` as "automatically installed everywhere."

## Public Template Boundary

Keep the template intentionally minimal.

It should usually contain:

- system files
- adapters
- bootstrap and migration helpers
- minimal manifests
- seed skills such as `manage-agent-skills`

It should not casually absorb:

- the user's private catalog
- large personal manifest sets
- one-off experiments that are not part of the product contract

## Canonical Layout

```text
skills/
  shared/
  antigravity/
  claude-code/
  codex/
  gemini-cli/
agents/
  antigravity.toml
  claude-code.toml
  codex.toml
  gemini-cli.toml
manifests/
  agent-global/
    antigravity.toml
    claude-code.toml
    codex.toml
    gemini-cli.toml
installs/
  agent-global/
    antigravity/
    claude-code/
    codex/
    gemini-cli/
templates/
  project-manifest.toml
bootstrap/
  install_agent_skills.py
  discover_global_skills.py
  apply_global_skill_migration.py
```

## Invariants

- `skills/shared/<skill>` is the canonical source for cross-agent skills.
- `skills/<agent>/<skill>` is the canonical source for agent-specific skills.
- `agents/<agent>.toml` is the compatibility contract for that harness's native global and project paths.
- `agents/<agent>.toml` also declares how that agent-global install should be materialized.
- `manifests/agent-global/<agent>.toml` declares only that agent's global install set.
- Agent-global manifests use explicit refs such as `shared/example-skill` or `codex/example-agent-skill`.
- Generated install views are outputs, not primary authoring surfaces.
- Migration V1 is agent-global only. Do not auto-scan or auto-import project-local skills during onboarding. Use the explicit `adopt-project` workflow after bootstrap when a project needs to bring local skills into the shared registry.
- Project installs are shared-only, declared in `.agent-skills.toml`, and materialized into `.agents/skills` plus any distinct native project mirrors such as `.codex/skills` and `.claude/skills`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yammaku/cross-agent-skills-template](https://github.com/yammaku/cross-agent-skills-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

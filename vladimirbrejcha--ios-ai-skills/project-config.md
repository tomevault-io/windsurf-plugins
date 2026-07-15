---
trigger: always_on
description: This repo is a collection of Codex skills. Checked-in local skills live in
---

# Agent Guide

This repo is a collection of Codex skills. Checked-in local skills live in
top-level directories with `SKILL.md` files; registry-covered external skills
point to their authoritative upstream sources through `skills.registry.yaml` and
`skills.lock.yaml`.

## Structure
- One folder per checked-in local skill at repo root.
- Every skill folder must include `SKILL.md` with YAML front matter (`name`, `description`).
- Optional folders: `assets/`, `scripts/`, `references/`.
- `skills.registry.yaml` is the sole disposition, source-ownership, and
  update-policy manifest. Every top-level `SKILL.md` must appear exactly once.
- `skills.lock.yaml` is reviewed lock/version metadata for resolved sources;
  unresolved and legacy inventory-only entries must not appear there.
- `provenance.sources.yaml` records public-safe upstream provenance
  observations and unresolved candidates for local skill folders.
- `skills.catalog.json` and `docs/skills-catalog.md` are generated public
  catalog views derived from registry, lock, the checked-in example profile,
  and `SKILL.md` metadata.
- `docs/registry-contract.md` is the public contract for source ownership,
  lock/version metadata, generated adapter views, public-safety requirements,
  and completion criteria.
- `docs/usage.md` contains public and 51Code operator workflows.
- `docs/setup-update-workflow.md` contains the public-safe new-machine,
  existing-machine, repo-local setup, verification, failure recovery, and
  restart workflow.
- `docs/contributing.md` contains skill editing, third-party update, fork, and
  validation workflows.
- `docs/manager-boundary.md` defines the accepted boundary between this public
  registry and the upstream `skills` CLI.
- `profiles/` may contain desired machine or repo exposure profiles for sync
  planning.
- `docs/` may contain public registry workflows and historical reports.
- `scripts/` may contain inventory, verification, doctor, and sync helpers.
- `scripts/skills_catalog.rb` generates and checks public catalog artifacts.
- `scripts/skills_sync.rb --plan` previews adapter create/update/remove actions
  without changing Codex, Claude, machine, or repo-local consumer folders.
- `scripts/skills_upstream_updates.rb` reports stale external-git pins and the
  evidence needed for reviewed update PRs; it does not mutate registry, lock,
  catalog, or adapter files.
- `scripts/skills_provenance_audit.rb` reports source-ownership drift,
  unregistered external imports, unresolved provenance candidates, and local
  duplicate skills; it does not fetch from the network or mutate sources.

## Local Operator Context

Use local operator-provided context if available. Do not commit private/local context files, machine paths, credentials, internal task links, or company-only notes.

## How to work in this repo
- If a task mentions a checked-in local skill, open that skill's `SKILL.md` and
  follow its workflow. If it mentions a registry-covered external skill, use the
  registry and lock metadata to find the reviewed upstream source instead of
  assuming a local `SKILL.md` exists.
- Use the front matter in `SKILL.md` as the source of truth for name/description.
- Use `skills.registry.yaml` as the source of truth for ownership, upstream
  source, update policy, and intended consumer exposure.
- Use `skills.lock.yaml` as the reviewed resolved-version input for sync plans.
- Do not edit generated catalog artifacts by hand. Update registry, lock, the
  checked-in example profile, or skill front matter, then run
  `scripts/skills_catalog.rb --write`.
- Active reusable skills must have one source owner, lock/version metadata, and
  generated adapter views for Codex, Claude Code, OpenCode, and repo-local
  consumers.
- Only active entries may emit install commands. Use `unresolved-local` with
  `needs-source-review` or `legacy` to record checked-in non-installable folders
  without making an ownership claim.
- Keep edits scoped to the requested skill(s); avoid cross-skill changes unless asked.
- When adding/removing a skill, update the README skills list and regenerate
  the catalog if registry-covered metadata changed.
- Do not edit imported consumer copies in `~/.codex/skills`, `~/.agents/skills`,
  `~/.claude/skills`, or product repo `.agents/skills`; update the owning skill
  source or registry manifest instead.
- Use pinned upstream `npx skills` commands for normal install/update/remove
  behavior when supported. Keep local scripts focused on policy checks,
  planning, and post-write verification.
- Run `scripts/skills_provenance_audit.rb --markdown` before resolving an
  unreviewed skill disposition or promoting a copied skill into a profile. Treat
  registry-local skills with reviewed external provenance as follow-up
  reclassification or fork decisions, not as silent source ownership.
- Use `scripts/skills_upstream_updates.rb --fail-on-stale` for scheduled or
  manual stale external-pin detection before preparing third-party update PRs.
- Use `docs/setup-update-workflow.md` as the canonical setup/update runbook;
  keep commands pinned, expected outcomes explicit, and unsupported adapter
  writes in manual review.
- Use `adapter: manager-copy` only in explicit reviewed profiles for targets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VladimirBrejcha/iOS-AI-Skills](https://github.com/VladimirBrejcha/iOS-AI-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->

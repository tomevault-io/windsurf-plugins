---
trigger: always_on
description: Context for agent sessions (Claude Code or Codex) working on this template repo. This is design rationale and structure — not user-facing docs (those are in `README.md` and `docs/`) and not marketplace-authoring procedures (those live in `plugins/marketplace-admin/skills/marketplace-manager/SKILL.md`).
---

# CLAUDE.md

Context for agent sessions (Claude Code or Codex) working on this template repo. This is design rationale and structure — not user-facing docs (those are in `README.md` and `docs/`) and not marketplace-authoring procedures (those live in `plugins/marketplace-admin/skills/marketplace-manager/SKILL.md`).

`AGENTS.md` is a one-line `@CLAUDE.md` import — Codex auto-resolves it, so this file is read by both runtimes. See [docs/adr/0001-parallel-manifests-shared-skills-tree.md](docs/adr/0001-parallel-manifests-shared-skills-tree.md) and [CONTEXT.md](CONTEXT.md) for the resolved domain language.

## What this repo is

A GitHub template repo that, when forked, becomes a private skills marketplace for a small team — usable from both Claude Code and Codex CLI. The owner forks → runs init → pushes → teammates install via `/plugin marketplace add` (Claude Code) or `codex plugin marketplace add` (Codex). Built on each runtime's native plugin marketplace primitive — no custom services, no extra infrastructure.

Target user: business owners and small-team operators who already use Claude Code or Codex individually and want their team to share skills without copy-pasting them in Slack.

## Structure

```
.claude-plugin/marketplace.json       # Claude Code marketplace registry
.agents/plugins/marketplace.json      # Codex marketplace registry
plugins/
  team-skills/                        # the plugin teammates install
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    skills/example-skill/             # ships as a smoke-test; owner deletes after first install works
  marketplace-admin/                  # owner-only plugin; not in teammate install instructions
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    skills/marketplace-manager/
      SKILL.md                        # init + import + add-plugin + publish + status (all flows inline)
CLAUDE.md                             # this file; auto-loaded by Claude Code
AGENTS.md                             # @CLAUDE.md; auto-loaded by Codex
CONTEXT.md                            # domain glossary
docs/
  adr/                                # architecture decision records
  install-claude-code.md              # teammate install guide (Claude Code)
  install-codex.md                    # teammate install guide (Codex)
README.md                             # marketing pre-init; init swaps it with README.instance.md
README.instance.md                    # post-init README skeleton with REPLACE_ME_* sentinels
LICENSE                               # MIT (init can switch to UNLICENSED or other)
.gitignore
```

## Key design decisions

### 1. Native marketplaces (both runtimes), not a custom layer
Both `/plugin marketplace add` (Claude Code) and `codex plugin marketplace add` (Codex) already solve discovery, install, update, enable/disable. Anything custom would re-implement what each vendor already ships. The "10 minutes" promise is only credible because we don't build infrastructure. We ship parallel hand-maintained manifests for both runtimes; see ADR-0001.

### 2. Two plugins out of the box: `team-skills` + `marketplace-admin`
- `team-skills` is what teammates install. Skills shared across the team go here.
- `marketplace-admin` is owner-only. It contains the `marketplace-manager` skill — the tooling the owner uses to administer the marketplace.

The owner installs `marketplace-admin` from their *own* marketplace (in whichever runtime they use — Claude Code, Codex, or both). This makes the management skill globally available — owner can trigger it from any session anywhere on disk, not just from inside the cloned repo.

We deliberately do **not** ship a project-scoped `.claude/skills/` skill at repo root. That would duplicate the marketplace-admin plugin and create "which one fires" ambiguity. The plugin is the single source of truth.

### 3. Single-plugin default, multi-plugin extensible
Day-zero config has one team-facing plugin (`team-skills`). When the team grows enough to want domain separation (`sales-skills`, `ops-skills`), the marketplace-manager skill's `add-plugin` flow scaffolds new plugins. We chose this default because:
- Single install command is honest about "10 minutes."
- Skills load progressively (only when triggered) — there's no per-skill cost to bundling many in one plugin.
- Splitting later is a directory move + marketplace.json edit, not a migration users feel.

### 4. Marketplace-manager is one SKILL.md, all flows inline
Five flows (init, import, add-plugin, publish, status) live in a single SKILL.md, not split across sub-files. The user explicitly chose this over progressive disclosure. Trade-off: every trigger pays the full token cost; in exchange, simpler authoring and one place to read.

### 5. Sentinels: `REPLACE_ME_*`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bradautomates/company-skills-marketplace-template](https://github.com/bradautomates/company-skills-marketplace-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->

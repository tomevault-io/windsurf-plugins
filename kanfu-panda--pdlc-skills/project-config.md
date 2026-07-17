---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

PDLC is a **Claude Code plugin**. It exposes 35 standardized "Product Development Life Cycle" stages as slash commands (`/pdlc-feature`, `/pdlc-prd`, `/pdlc-tdd`, ..., `/pdlc-onboard`) covering PRD → Design → TDD → Implement → Review → Ship → Deploy → Retro and 21 specialized tools.

The repo is **both a plugin and a single-plugin marketplace** (so `claude plugin marketplace add github:kanfu-panda/pdlc-skills` registers it directly).

This plugin is **Claude Code only** — it relies on Claude Code's plugin / skill mechanism. There's no port to Cursor, Copilot, Cline, etc.

## Repository layout

```
pdlc-skills/
├── .claude-plugin/
│   ├── plugin.json                 ← plugin manifest (name, version, author, ...)
│   └── marketplace.json            ← marketplace manifest (so the repo is also a marketplace)
├── skills/                         ← 35 sub-skills (each = one slash command)
│   ├── pdlc-feature/SKILL.md       → /pdlc-feature
│   ├── pdlc-prd/SKILL.md           → /pdlc-prd
│   ├── pdlc-tdd/SKILL.md           → /pdlc-tdd
│   └── ... (35 dirs total)
├── references/
│   └── templates/
│       ├── *-template.md           ← 9 user-facing document templates
│       └── prompts/*.md            ← 9 shared prompt fragments (iron-law / handoff / ...)
├── install.sh                      ← curl-based one-line installer wrapping `claude plugin install`
├── docs/
│   └── usage-guide.md              ← single user manual (architecture + reference + scenarios)
├── tests/
│   ├── frontmatter-check.sh        ← validates skills/<name>/SKILL.md frontmatter
│   └── install-smoke.sh            ← end-to-end install layout test
└── VERSION                         ← canonical version (mirrored in plugin.json)
```

## Sub-skill naming

Every sub-skill at `skills/pdlc-<name>/SKILL.md` becomes the slash command `/pdlc-<name>` in Claude Code. The `pdlc-` prefix is **part of the skill name**, not a namespace separator. We chose this over the colon namespace `/pdlc:<name>` for two reasons:

1. Visual distinctiveness — typing `/pdlc-` filters cleanly to all 35 PDLC commands; suffix-only names (`/feature`, `/fix`) collide with built-in commands and other plugins.
2. Backwards compatibility — matches the v1 mental model of `/pdlc-feature`.

The full plugin namespace is `pdlc:pdlc-<name>` formally, but Claude Code's autocomplete simplifies to `/pdlc-<name>` since the suffix is unique. Both invocations route to the same skill.

## Common commands

Install / upgrade / uninstall (uses Claude Code's `claude plugin` CLI under the hood):

```bash
# One-line curl install
curl -fsSL https://raw.githubusercontent.com/kanfu-panda/pdlc-skills/main/install.sh \
  | bash -s -- --global

# Or, equivalent native commands
claude plugin marketplace add kanfu-panda/pdlc-skills
claude plugin install pdlc@pdlc-skills
```

For local development from a clone:

```bash
claude plugin marketplace add /Users/me/projects/pdlc-skills
claude plugin install pdlc@pdlc-skills
```

Tests (run via GitHub Actions on every PR; can also be run manually):

```bash
bash tests/frontmatter-check.sh   # validate skills/*/SKILL.md frontmatter
bash tests/install-smoke.sh       # end-to-end install layout assertions
shellcheck install.sh tests/*.sh  # bash linting
```

## How sub-skills compose

Each `skills/pdlc-<name>/SKILL.md` has:

- YAML frontmatter (`name`, `description`, `argument-hint`, `allowed-tools`, plus PDLC-internal fields `layer`, `stage`, `produces`, `requires`, `next_step`, `terminal_state`)
- Markdown body — the workflow Claude follows when the slash command fires
- `<!-- @include templates/prompts/<x>.md -->` directives — shared prompt fragments (IRON LAW, handoff, self-audit, etc.) that Claude inlines from `references/templates/prompts/<x>.md` at runtime

The `@include` mechanism is **not** preprocessed by Claude Code — it relies on Claude reading the comment and following it on demand. This works in practice but is not a documented Claude Code feature.

## Layer structure

Sub-skills are grouped by `layer:` in frontmatter (the 35 names below all carry the `pdlc-` prefix):

- **Layer 1 (3)**: `pdlc-feature`, `pdlc-fix`, `pdlc-status` — one-sentence-driven entry points
- **Layer 2 (11)**: `pdlc-prd`, `pdlc-design`, `pdlc-tdd`, `pdlc-implement`, `pdlc-review`, `pdlc-e2e`, `pdlc-refactor`, `pdlc-ship`, `pdlc-deploy`, `pdlc-retro`, `pdlc-task` — single-stage fine control
- **Layer 3 (21)**: specialized tools (`pdlc-ui-design`, `pdlc-db-design`, `pdlc-arch`, `pdlc-lint`, `pdlc-perf`, `pdlc-security`, `pdlc-code-gen`, `pdlc-add-service`, `pdlc-add-app`, `pdlc-api-mock`, `pdlc-db-migrate`, `pdlc-i18n`, `pdlc-changelog`, `pdlc-standard`, `pdlc-relate`, `pdlc-bootstrap`, `pdlc-adopt`, `pdlc-onboard`, `pdlc-ui-design-pro`, `pdlc-loop-next`, `pdlc-loop-run`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kanfu-panda/pdlc-skills](https://github.com/kanfu-panda/pdlc-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->

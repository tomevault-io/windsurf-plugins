---
trigger: always_on
description: Use when migrating Meteor 2.x code to Meteor 3.x async APIs. Triggers on
---

# Authoring an Agent Skill for `meteor/agent-skills`

This document is the contract for everyone writing a skill in this repo. Plan 01 created it; subsequent plans must not break it without updating it here first.

## Maintainer skills

Repository maintenance workflows live under `.github/skills/`. They help contributors maintain the published catalog but are not included in release ZIPs or the README catalog.

| Skill | Use when |
|-------|----------|
| [`skill-maintenance`](.github/skills/skill-maintenance/SKILL.md) | Creating, reviewing, or updating a published skill while preserving repository conventions. |
| [`skill-gap-audit`](.github/skills/skill-gap-audit/SKILL.md) | Comparing Meteor documentation and source changes with current skill coverage. |
| [`release-publishing`](.github/skills/release-publishing/SKILL.md) | Preparing, testing, tagging, or publishing beta and stable plugin releases. |

### Audit and maintenance workflow

`skill-gap-audit` is read-only by default. It produces an evidence-backed report and does not change published skills. Confirmed findings move to `skill-maintenance` only when the user requests implementation.

```text
skill-gap-audit
-> confirmed findings
-> authorized implementation scope
-> skill-maintenance
-> affected manual cases when routing or high-risk guidance changes
-> repository validation
-> release ZIP verification
```

When one request explicitly asks to audit and fix, complete the audit first, preserve its evidence, then load `skill-maintenance` and implement only confirmed findings within the requested scope. Do not implement uncertain findings or create proposed skills without explicit user approval.

`pnpm run validate` enforces frontmatter, naming, body size, prohibited-content rules, required evaluation-case files, plugin packaging consistency, and required revision and handoff fields in committed gap-audit reports. `pnpm run check-links` checks links in published and maintainer skills. The ZIP test suite verifies that `.github/skills/` and repository maintenance evidence remain outside release artifacts.

Committed gap-audit records live under `audits/skill-gaps/`. Each record identifies the audited agent-skills revision, Meteor remote and revision, release context, audit mode, and previous audit baseline. These reports are immutable maintenance evidence and are never included in distributable skills. An incremental audit uses the latest applicable committed record from Git history; if no reliable record exists, run a full audit.

Each published skill keeps its complete manual acceptance cases in `references/eval-cases.md`. These cases are the single source of truth for behavior checks. Run affected prompts in fresh conversations or disposable projects and record pass or fail outcomes in the PR or review. Do not commit temporary projects, transcripts, or raw model output. CI validates that the case file exists but does not invoke a model.

For a human-oriented explanation of how source review, deterministic checks,
manual behavior checks, and ZIP inspection fit together, read
the [maintenance verification guide](docs/maintenance-verification.md).

## What is a skill

A skill is a folder under `skills/` that helps an AI coding assistant work on Meteor 3 applications. A skill has:

```
skills/<name>/
├── SKILL.md             # required
├── references/          # optional: longer reference Markdown
├── scripts/             # optional: helper bash or .mjs scripts
└── assets/              # optional: templates, fixtures, images
```

`<name>` is lowercase-kebab-case, `[a-z0-9-]{1,64}`, equal to the `name` field in the frontmatter.

## Frontmatter

`SKILL.md` is YAML frontmatter followed by a Markdown body. The frontmatter is validated against `skill.schema.json`.

Required:

```yaml
---
name: meteor-async-migration
description: >
  Use when migrating Meteor 2.x code to Meteor 3.x async APIs. Triggers on
  callAsync, findOneAsync, removed Fibers, "X is not a function" errors after
  upgrade. Ask about top-level await and async cursors.
metadata:
  author: meteor
  kind: knowledge
  meteor: ">=3.0"
  area: migration
  tagline: "Migrate Meteor 2.x code to Meteor 3.x async APIs (callAsync, findOneAsync, Fibers removal)."
---
```

`description` is agent-facing and packed with trigger phrases so the assistant picks the skill against a user prompt. State the outcome, concrete positive triggers, and a boundary or handoff when neighboring skills overlap. Test routing changes with positive and near-miss prompts in the affected skills' `references/eval-cases.md`. `tagline` is human-facing: a short one-liner (16-200 chars) rendered as the bullet text in the README catalog. Keep them separate; do not collapse one into the other.

Optional:

```yaml
metadata:
  bundle: ["migration", "essentials"]
  docs_synced_at: "2026-05-14"
license: MIT
```

### Rules the validator enforces

- `name` must equal the folder name.
- `description` is <=1024 characters and contains at least two trigger phrases. Trigger phrases include: `Use when`, `Use this skill when`, `Use this Skill when`, `Triggers on`, any `ask about` or `asks about` substring.
- `metadata.tagline` is 16-200 characters. Rendered verbatim into the README catalog.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meteor/agent-skills](https://github.com/meteor/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->

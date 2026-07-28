---
trigger: always_on
description: Specs-driven development workflow with design system integration
---


# Specs-Driven Development Workflow

This project uses a specs-driven workflow. The `.specs/` directory contains:
- `strategy.md` - Business strategy: target customer, buying motion, value prop
- `constitution.md` - Non-negotiable constraints: security, data handling, errors
- `migrations.md` - Database migration playbook: tool, conventions, reversibility (created by /infer-migrations)
- `features/` - Gherkin feature specifications with ASCII mockups
- `personas/` - User personas that inform every spec
- `test-suites/` - Documentation of what each test file covers
- `design-system/` - Design tokens and component patterns
- `learnings/` - Cross-cutting patterns by category
- `mapping.md` - Links features ↔ tests ↔ components ↔ design

---

## Command Triggers

When the user says any of these phrases, **automatically invoke `/spec-first`**:

| User says | Action |
|-----------|--------|
| "spec first" | Run `/spec-first {feature}` |
| "spec-first" | Run `/spec-first {feature}` |
| "write a spec for" | Run `/spec-first {feature}` |
| "create a spec" | Run `/spec-first {feature}` |
| "spec this out" | Run `/spec-first {feature}` |
| "spec out" | Run `/spec-first {feature}` |
| "plan this feature" | Run `/spec-first {feature}` |
| "write the spec" | Run `/spec-first {feature}` |
| "create spec" | Run `/spec-first {feature}` |
| "update the spec for" | Run `/spec-first {feature}` (update mode) |
| "update spec" | Run `/spec-first {feature}` (update mode) |

When the user says any of these after a spec is shown, **invoke `/tdd`**:

| User says | Action |
|-----------|--------|
| "tdd" | Run `/tdd {feature}` |
| "go ahead" | Run `/tdd` with current spec |
| "build it" | Run `/tdd` with current spec |
| "implement it" | Run `/tdd` with current spec |
| "ship it" | Run `/tdd` with current spec |

Extract the feature description from the rest of their message.

When the user says any of these, **invoke the corresponding ralph/utility command**:

| User says | Action |
|-----------|--------|
| "ralph setup", "set up ralph", "configure ralph" | Run `/ralph-setup` |
| "ralph run", "run ralph", "start the loop", "run the loop" | Run `/ralph-run` |
| "clean slate", "kill everything", "restart everything", "nuke localhost" | Run `/clean-slate` |
| "generate guide", "update guide", "how to use guide" | Run `/guide` |

When the user says any of these, **invoke `/strategy`**:

| User says | Action |
|-----------|--------|
| "strategy" | Run `/strategy` |
| "product strategy" | Run `/strategy` |
| "business strategy" | Run `/strategy` |
| "shape this" | Run `/strategy` |
| "who are we selling to" | Run `/strategy` |
| "business model" | Run `/strategy` |

When the user says any of these, **invoke `/gtm`**:

| User says | Action |
|-----------|--------|
| "gtm playbook" | Run `/gtm` |
| "marketing plan" | Run `/gtm` |
| "how do we get users" | Run `/gtm` |
| "distribution plan" | Run `/gtm` |
| "growth plan" | Run `/gtm` |
| "channel strategy" | Run `/gtm` |
| "launch plan" | Run `/gtm` |
| "outreach plan" | Run `/gtm` |

When the user says any of these, **invoke `/find-early-users`**:

| User says | Action |
|-----------|--------|
| "find early users" | Run `/find-early-users` |
| "find users" | Run `/find-early-users` |
| "find prospects" | Run `/find-early-users` |
| "find people" | Run `/find-early-users` |
| "who should I talk to" | Run `/find-early-users` |
| "find beta testers" | Run `/find-early-users` |
| "find feedback" | Run `/find-early-users` |
| "prospect list" | Run `/find-early-users` |
| "who's complaining about" | Run `/find-early-users` |
| "find my first users" | Run `/find-early-users` |

When the user says any of these, **invoke `/constitution`**:

| User says | Action |
|-----------|--------|
| "constitution" | Run `/constitution` |
| "project constraints" | Run `/constitution` |
| "security rules" | Run `/constitution` |
| "invariants" | Run `/constitution` |
| "non-negotiables" | Run `/constitution` |
| "audit specs" | Run `/constitution --audit` |

When the user says any of these, **invoke `/infer-migrations`**:

| User says | Action |
|-----|-----|
| "infer migrations" | Run `/infer-migrations` |
| "migration strategy" | Run `/infer-migrations` |
| "migration playbook" | Run `/infer-migrations` |
| "how do we do migrations" | Run `/infer-migrations` |
| "document our migrations" | Run `/infer-migrations` |
| "schema change strategy" | Run `/infer-migrations` |

**Create vs Update**: `/spec-first` auto-detects whether to create or update: searches `.specs/features/` for a matching spec (by path or frontmatter `feature:`). Match found → update existing spec. No match → create new spec. With `--full`, both paths continue through the full Red-Green-Refactor TDD cycle.

**Specs are state, not deltas**: a spec always describes the entire expected behavior of the feature as it exists now — "add three fields to the form" is a commit message, not a spec. Updates rewrite affected scenarios to the new truth (removing superseded ones) instead of appending change notes; the delta lives in git history.

### Full Mode Triggers

If user includes "full", "auto", "no stops", or "don't pause":
- Add `--full` flag to the command
- Example: "spec first user auth, full mode" → `/spec-first user auth --full`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdrianRogowski/auto-sdd](https://github.com/AdrianRogowski/auto-sdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->

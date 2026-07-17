---
trigger: always_on
description: This file tells AI agents how to load and use skills from this repository. Skills in this repo follow the [Agent Skills open format](https://agentskills.io) — a standardized way to give agents new capabilities through structured markdown files.
---

# AGENTS.md — Agent Guide for agent-skills

This file tells AI agents how to load and use skills from this repository. Skills in this repo follow the [Agent Skills open format](https://agentskills.io) — a standardized way to give agents new capabilities through structured markdown files.

## Format Compliance

Every skill in this repository conforms to the [Agent Skills specification](https://agentskills.io/specification.md):

| Requirement | Rule |
|-------------|------|
| Directory | Each skill in its own directory named by the skill |
| Entry point | `SKILL.md` with YAML frontmatter + markdown body |
| `name` field | Lowercase, hyphens only, matches parent directory name |
| `description` field | Trigger-oriented, describes what and when |
| Progressive disclosure | Core instructions in `SKILL.md` (< 500 lines, < 5,000 tokens), supporting material in `references/`, `templates/`, `scripts/` |
| File references | Relative paths from skill root, one level deep |
| **Human-readable README** | **`README.md`** in skill root — required for every skill. See [README Format](#readme-format) below |

## README Format

Every skill directory **MUST** contain a `README.md` written for a **human audience** (not an AI agent). The README explains what the skill does and why someone would want to install it. It is the public face of the skill — the first thing a human sees when browsing the repository.

### Required Sections

| Section | Purpose |
|---------|---------|
| **Title** | Skill name + one-line summary of what it does |
| **Why Install This Skill** | 2-3 paragraph pitch answering "what problem does this solve for me?" and "what can my agent do after installing this?" — written in plain language, not format docs |
| **What You Get** | Table listing directory contents (scripts, references, templates, assets) and what each provides |
| **Quick Start** | Minimal setup: env vars to export, first command to run (omit for reference-only skills) |
| **Triggers** | List of trigger conditions that tell someone when to load this skill |
| **Requirements** | Dependencies, API keys, Python version, system tools |

### Style Guidance

- **Lead with benefit, not implementation.** Answer "what does this do for me?" before "what tech is it built on?"
- **Be concrete.** Show real command examples with expected output. Avoid abstract descriptions.
- **Assume the reader is human.** No agent instructions, no JSON schemas, no progressive disclosure notes. Those go in `SKILL.md`.
- **Keep it scannable.** Use tables, code blocks, and bullet lists. A human should grasp the skill's purpose in 10 seconds.
- **One page or less.** A README that takes more than a minute to read is too long. Save depth for `SKILL.md`.

### Example

See [data-scientist/README.md](data-scientist/README.md) or any skill in this repository for the canonical format.

## State-Modifying Skills

Skills that change external state must say so explicitly and use this gate before the first mutation:

> Confirm the target, scope, and rollback path before acting. Read-only discovery may proceed without confirmation.

Destructive operations still require an explicit user directive; this convention does not authorize deletion, privilege changes, or irreversible cleanup.

## Failure-Mode Routing

For problem-pattern routing, start with [FAILURE-MODE-INDEX.md](FAILURE-MODE-INDEX.md).

## How to Load Skills

Skills are loaded progressively in three stages:

### Stage 1 — Metadata

At session start, read each skill's `name` and `description` from frontmatter. This takes ~100 tokens per skill and lets you know what's available without loading full content.

```yaml
# Example metadata (from cli-builder/SKILL.md)
name: cli-builder
description: >-
  Build or refactor CLI tools designed for AI agent consumption: non-interactive,
  flag-driven, idempotent, with --json output and --dry-run preview.
```

### Stage 2 — Full Instructions

When a user's request matches a skill's description keywords, load the full `SKILL.md`. The body contains step-by-step instructions, examples, and gotchas. Do not load skills preemptively — only load when triggered.

### Stage 3 — Supporting Files

Reference files (`references/`, `templates/`, `scripts/`) are loaded on demand. The `SKILL.md` tells you when to read each one. Do not load all references at activation time — following the triggers preserves context.

## Reading Order

If this is your first session with this repo, read these in order:

1. [agent-skills/SKILL.md](agent-skills/SKILL.md) — The Agent Skills format reference. Read this first to understand the format.
2. [README.md](README.md) — Skill index with descriptions. Use to discover which skill to load.
3. Individual skill `SKILL.md` files as triggered by the user's task.

## Skill Loading by Trigger

When the user mentions these keywords, load the corresponding skill:

| User says... | Load skill |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magnus919/agent-skills](https://github.com/magnus919/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->

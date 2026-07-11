---
trigger: always_on
description: <!-- GENERATED from AGENTS.md by scripts/build-plugins.mjs — do not edit. -->
---

<!-- GENERATED from AGENTS.md by scripts/build-plugins.mjs — do not edit. -->
<!-- Edit AGENTS.md and run `npm run build:plugins`. -->

# Gemini CLI context — LEAN AI-SDLC harness

In Gemini CLI the skills below are invoked as `/lean:<name>` (e.g. `/lean:tdd`, `/lean:reviewer`). The full delivery workflow lives in WORKFLOW.md.

---

# AGENTS.md

Operating rules for any AI agent (Claude Code, Cursor, Copilot, Aider, or any
other tool that reads `AGENTS.md`) working in this repository. These rules
implement the [workflow](./WORKFLOW.md) and are deliberately strict — the harness
trades a little ceremony for reliable quality at agent speed.

The workflow is **human-in-the-loop (HITL)** and **non-subagentic** by design: a
human drives and reviews every stage; skills assist, but no autonomous subagent
owns a stage end-to-end. Read [`WORKFLOW.md`](./WORKFLOW.md) before starting work.

## Prime directives

1. **Follow a flow.** Every change starts at an entry point: `/discovery`
   (feature), `/bug-analysis` (bug), or `/goal` (change request). Don't skip to
   coding.
2. **Be LEAN.** Maximize the work _not_ done. Prefer the smallest change that
   satisfies the acceptance criteria. Reuse existing code — search before adding.
3. **Build quality in.** No production code without a failing test demanding it
   (`/tdd`). Keep the suite green at every commit (`GEN-004`, `GEN-005`).
4. **Respect the architecture.** Honor the ADRs in
   [`.archgate/adrs/`](./.archgate/adrs/). The archgate (`npm run archgate`) must
   stay green. New boundaries require a new ADR — author it with `/adr-author`.

## Language

The primary language of this project is **English**. All code, documentation, and
communication are in English.

**Written artifacts MUST be in English regardless of the language of the user
prompt.** This applies to every file, every commit message, and every code
comment. The only exception is content that is part of the product itself (e.g.
user-facing UI strings, translation files, domain terminology that has no
established English equivalent). When in doubt, write English.

ADRs in `.archgate/adrs/` and any other governance or process documentation
(READMEs, PRDs, plans, ADRs themselves) MUST be authored in English. The
product-content exception above does **not** extend to these files.

## Skills Layout

Agent skills live in `.agents/skills/<name>/SKILL.md` as the single source of
truth. `.claude/skills/<name>` must be a symlink pointing to
`../../.agents/skills/<name>` so every agent tool on the team sees the same skill
set.

When adding a new skill:

1. Create it under `.agents/skills/<name>/` (never directly under `.claude/skills/`).
2. Create the symlink: `ln -s ../../.agents/skills/<name> .claude/skills/<name>`.
3. Commit both the skill directory and the symlink.

When removing a skill, delete both the real directory and the symlink.

`scripts/check-skill-symlinks.sh` (`npm run check:skills`) enforces this
invariant and runs in the pre-push hook and CI. Symlinks require
`git config core.symlinks=true` (default on macOS/Linux); Windows development is
not supported, so no copy-based fallback is maintained.

## Rules Layout

Agent workspace rules (persistent instructions loaded automatically on startup)
live in `.agents/rules/<name>.md` as the single source of truth, mirroring the
skills layout. `.claude/rules/<name>.md` must be a symlink pointing to
`../../.agents/rules/<name>.md`.

When adding a new rule:

1. Create it under `.agents/rules/<name>.md` (never directly under `.claude/rules/`).
2. Create the symlink: `ln -s ../../.agents/rules/<name>.md .claude/rules/<name>.md`.
3. Commit both the rule file and the symlink.

When removing a rule, delete both the real file and the symlink.

`scripts/check-rule-symlinks.sh` (`npm run check:rules`) enforces this invariant
and runs in the pre-push hook and CI. The rule files are short ADR routers
(`general-adrs.md`, `architecture-adrs.md`) plus `styling-consistency.md`; they
point agents to the binding ADRs by domain rather than duplicating their content.

## Skill & Rule Authoring

Skills and workspace rules are shared by every agent on the team. They MUST be
written **agent-agnostic** and MUST NOT reference a specific agent vendor,
product, or model (e.g. "Claude", "Claude Code", "GPT", "Copilot", "Cursor").
Write from the team's perspective — use neutral terms like "the agent", "the
team", "you", or simply describe the task in the imperative. This applies to the
`description` frontmatter, body copy, examples, and any bundled reference files.
When updating an existing skill or rule, remove any vendor-specific phrasing you
encounter. The meta-skill `/write-better-skill` documents how to author skills
for this harness.

Each `SKILL.md` opens with YAML frontmatter. Declare only the tools the skill
actually uses:

```yaml
---
name: <name>
description: <one line — when to use>
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(archgate:*), Edit, Write, Agent
user-invocable: <true|false>
---
```

**No autonomous subagents.** Skills assist a human-driven stage; they do not
delegate a whole stage to an autonomous subagent. A human stays in the loop at
each stage (see [`WORKFLOW.md`](./WORKFLOW.md)).

## Plugin distribution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stefanstelzer/lean-harness-ai-sdlc](https://github.com/stefanstelzer/lean-harness-ai-sdlc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->

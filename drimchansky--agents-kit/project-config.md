---
trigger: always_on
description: This file is for **agents and humans working on this repo**. If you're an agent invoked from a _consumer_ project that has installed the kit, this file does not apply to you — your rules come from `./AGENTS.md` inside the skill directory you're running, which resolves to `CORE_RULES.md` at the kit root.
---

# Contributing to agents-kit

This file is for **agents and humans working on this repo**. If you're an agent invoked from a _consumer_ project that has installed the kit, this file does not apply to you — your rules come from `./AGENTS.md` inside the skill directory you're running, which resolves to `CORE_RULES.md` at the kit root.

## What this repo is

A kit of skills and shared rules distributed to coding agents (Claude Code, Codex, and others). It ships in two ways:

- **Claude Code plugin marketplace (preferred for end users)** — the repo carries both `.claude-plugin/plugin.json` (plugin manifest) and `.claude-plugin/marketplace.json` (single-entry marketplace whose plugin is sourced from the same repo via `"source": "./"`). End users install with `/plugin marketplace add drimchansky/agents-kit` followed by `/plugin install agents-kit@drimchansky-agents-kit`. To add a future plugin, append a new entry to `marketplace.json`'s `plugins` array with its own schema-valid `source` — no separate marketplace repo needed.
- **Claude Code `--plugin-dir`** — local-dev install against an unreleased clone: `claude --plugin-dir /path/to/agents-kit`.
- **`setup.sh` (manual / non-plugin)** — copies `skills/` and `references/` into each supported agent's home directory (`~/.claude/...`, `~/.codex/...`).

`README.md` covers user-facing structure and installation. This file covers **how to work on the kit**.

## Skill categories

The kit ships two categories of skill, and the Core Rules contract applies to only one of them:

- **Engineering-workflow skills** — operate on code and participate in the understand → plan → implement → review → verify → document loop. They carry the shared rules contract (symlink + directive, see below). Today, in workflow order: `explore`, `refine-idea`, `resume-task`, `plan-task`, `review-plan`, `implement-plan`, `review-code`, `verify-issue`, `review-docs`. Preserve the workflow ordering when listing them in docs and when inserting new ones (e.g. a new review-stage skill goes near `review-code`, not at the alphabetical end).
- **Standalone skills** — prose tools and single-purpose utilities that don't touch code and don't follow the engineering loop. They are deliberately self-contained: their `skills/<name>/` holds **only `SKILL.md`**, with **no `AGENTS.md` symlink** and **no Core Rules directive**. All guidance lives inline in `SKILL.md`. Today: `proofread`, `translate`, `fact-check`.

When in doubt, default to the engineering shape — adding the symlink + directive is cheap; retrofitting later is annoying.

## The Core Rules contract (engineering skills only)

The kit's shared rules live in **`CORE_RULES.md`** at the repo root. They're distributed to consumer projects through a per-skill mechanism — not as a global rules file. The contract applies to **engineering-workflow skills**; standalone skills are exempt by design (see "Skill categories" above).

- Each engineering skill's `skills/<name>/AGENTS.md` is a **relative symlink** to `../../CORE_RULES.md`.
- Each engineering skill's `skills/<name>/SKILL.md` opens with a fixed **"Core Rules" directive** that instructs the agent to:
    1. Read the sibling file `./AGENTS.md`.
    2. Apply the rules for the duration of the skill.
    3. Output `✅ Core agents-kit@<version> rules applied` on its own line, before any other text or tool calls. The `<version>` placeholder is substituted at runtime with the value on the **Version** line at the top of `CORE_RULES.md` (which the symlink resolves to).

**Don't break this contract.** When you add or edit an engineering skill:

- The directive block must be present and unmodified at the top of `SKILL.md`, between the closing `---` of the frontmatter and the existing body.
- The sibling `skills/<name>/AGENTS.md` must exist and point at `../../CORE_RULES.md`.
- The check-mark confirmation line is the user's only signal that rules were loaded; if it changes wording, every engineering skill must be updated together. The `<version>` placeholder in the source line stays literal — agents substitute the actual version from CORE_RULES.md at output time.
- **Bumping the kit version:** update `.claude-plugin/plugin.json` and the **Version** line near the top of `CORE_RULES.md` together. They are the two sources of truth; drift between them means the confirmation line shows a stale version, and CI fails the version-sync check (see "Verifying changes" below) so the bump can't merge until they match. Nothing else needs to change on a version bump — the `<version>` placeholders in skills are stable across releases.

If you want to change the rules themselves, edit `CORE_RULES.md`. The change propagates to all engineering skills automatically — symlinks resolve at read time. There is no build step.

## Symlink caveat


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drimchansky/agents-kit](https://github.com/drimchansky/agents-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

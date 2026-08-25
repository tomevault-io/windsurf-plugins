---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A multi-platform AI plugin (Claude Code, Cursor, Codex, and 9 other AI coding tools via a CLI installer) that adds Salesforce-aware compound engineering workflows. The product is the markdown under `skills/` — those files are what get distributed and what other AI clients load. The TypeScript CLI under `cli/` translates that markdown into each target tool's expected directory layout and hosts the optional local Tend feed runtime.

A Salesforce developer using this plugin gets nine workflow entry points (ideate → brainstorm → plan → deepen → work → review → polish → compound, plus `sf-lfg` for the full pipeline) and the Tend-style `/sf-tend` responsibility-feed layer. The conditional **polish** phase (SLDS2/UX, accessibility, copy — UI work only) runs via the stack-aware `sf-polish` skill. The whole thing is backed by 61 specialist **personas** — prompt assets the workflow skills dispatch as isolated subagents (parallel on Claude Code, inline on harnesses without a subagent primitive) — plus domain-knowledge skills (governor limits, Apex/LWC/Flow patterns, security, integrations, Agentforce, hosted MCP). Ideate and polish are the human "bread"; the middle stages are the AI loop. An optional repo-root `STRATEGY.md` (created and maintained by `/sf-strategy`) grounds ideate/brainstorm/plan when present.

**Agentless (V3.1).** The plugin ships **no standalone registered agents** — formal agent definitions are not a reliable common denominator across Claude, Codex, Cursor, Gemini, Pi, OpenCode, etc. Specialist behavior lives as skill-local **persona prompt assets** under `skills/<owner>/references/personas/<name>.md`, owned by the workflow skill that dispatches them and referenced cross-skill by relative path. They ship to every platform as ordinary skill files (the CLI copies a skill's whole subtree).

## Principles are the source of truth

`PRINCIPLES.md` lists seven numbered principles that govern every skill, every persona, and every code review. README, this file, and the nine workflow skills reference them **by number** rather than restating them. When something in this repo (a skill, an agent, a PR) conflicts with a principle, fix the implementation — don't soften the principle. When principles conflict with each other (rare), the lower-numbered principle wins.

Read `PRINCIPLES.md` before making non-trivial changes to workflow skills.

## Protected directories — never delete or overwrite

* `docs/solutions/` — institutional knowledge, irrecoverable. Cumulative across the project's lifetime.

* `docs/plans/` — implementation plans are permanent project records.

* `docs/brainstorms/` — pre-planning exploration records.

If content is wrong, **edit** it. If it's obsolete, add `status: deprecated` to the YAML frontmatter. Do not `rm`. Do not `git filter-repo` these paths.

## Architecture

* **Skills** live at `skills/<name>/SKILL.md` and are the user-facing entry points. They auto-route from natural-language phrases via the `description` frontmatter; direct invocation (`/sf-<name>`) also works. V3 retired the `commands/` directory entirely — skills replaced commands.

* **Personas** are skill-local prompt assets at `skills/<owner>/references/personas/<name>.md` — **not** registered agents (V3.1 went agentless). Each is owned by the workflow skill that dispatches it; primary owners are `sf-review` (code-review personas), `sf-doc-review` (doc-review personas), and `sf-plan` (research personas). Other skills reference a persona by relative path (e.g. `../sf-review/references/personas/<name>.md`) — stable because the whole `skills/` tree ships together.

* **Workflow skills dispatch personas as isolated subagents.** `sf-review`, `sf-work`, `sf-doc-review`, and `sf-lfg` load the persona file's contents and feed them to a general-purpose subagent via the Task tool — parallel with isolated context on Claude Code, applied inline in sequence on harnesses without a subagent primitive. When adding a new review concern, add a persona file under the owning skill's `references/personas/` and wire its name into that skill's dispatch list.

* **Multi-platform manifests**: `.claude-plugin/`, `.cursor-plugin/`, `.codex-plugin/`. Each carries per-platform plugin metadata; the CLI in `cli/` reads the canonical source and writes the others.

## Frontmatter conventions

* **Skills**: `name`, `description`, `argument-hint`. The `description` field is what powers auto-routing — enumerate Salesforce-flavored trigger phrases there, not in the body.

* **Personas** (`references/personas/<name>.md`): minimal frontmatter — `name`, `description` only. They are prompt assets, not registered agents, so the agent-era `model`, `tools`, `color`, and `scope` fields are gone. The dispatching skill chooses the subagent's tools/model at dispatch time; review/research personas are read-only by intent, and the four that write files (`sf-bug-reproduction-validator`, `sf-pr-comment-resolver`, `sf-deployment-verification-agent`, `sf-mcp-tool-builder-agent`) say so in their prose.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [divingsbysangam/salesforce-compound-engineering-plugin](https://github.com/divingsbysangam/salesforce-compound-engineering-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->

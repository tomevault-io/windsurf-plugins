---
trigger: always_on
description: Root entry point for all agents operating in this repository. Defines session-start loading protocol, repository layout, automation surface, non-negotiable editing rules, and security/GDPR constraints. Read on every session before any other instruction.
---

<!--
updated: 2026-04-18
-->

# AGENTS.md

<role>
Root entry point for all agents operating in this repository. Defines session-start loading protocol, repository layout, automation surface, non-negotiable editing rules, and security/GDPR constraints. Read on every session before any other instruction.
</role>

<summary>
> Personal work journal organized as Markdown files under **PARA** (Projects, Areas, Resources, Archive) with supporting shell scripts. Operates on **progressive disclosure**: AGENTS.md carries the rules, skills carry workflows, context files carry project-specific detail. Load only what the current task requires.
</summary>

<inputs>

| Input | Source | Required |
|---|---|---|
| Author identity, team, tooling | `context/context.md` | Yes (Tier 0) |
| Resource articles | `resources/` via `qmd query` | When task touches writing/people/teams/projects (Tier 1) |
| Codebase context | `codebases/<name>.md` | When editing external repo (Tier 2) |

</inputs>

<definitions>

- **PARA** — Projects, Areas, Resources, Archive. Top-level organization scheme.
- **Progressive disclosure** — Load skills, workflows, and context files on demand, not upfront.
- **Tier 0/1/2** — Session-start loading stages, scoped to task type.
- **Stub** — Minimal placeholder resource article created when an entity is referenced but has no article yet.

</definitions>

<tiers>

**Tier 0 — every session**
DO read `context/context.md` (author identity, team, active projects, tooling). Skip if already loaded this session.

**Tier 1 — task involves writing, resources, people, teams, or projects**
DO search `resources/` with `qmd query "<topic>"` or browse the directory tree to identify articles relevant by topic, team, person, or project tag. DO read specific resource articles that directly bear on the task. If an article should exist but doesn't, DO create a stub before proceeding.

**Tier 2 — coding work in external repos**
DO read `context/context.md` `## Codebases` to identify the active codebase. DO read `codebases/<name>.md` — it contains architecture, tech stack, build commands, test commands, coding conventions, CI context, and key decisions. If no `codebases/<name>.md` exists, DO create one before proceeding. NEVER invent conventions. If the file is missing or a field is empty, ASK the user before assuming.

</tiers>

<pre_check>

1. **Context loaded** — `context/context.md` read this session. If empty or missing, direct user to run `/bootstrap` before any context-dependent task.
2. **Task tier identified** — Determine whether Tier 1 and/or Tier 2 loading applies.
3. **Context currency** — If the user shares new project, role change, team update, tool adoption, or any fact belonging in `context/context.md` or `codebases/<name>.md`, update the file immediately during the session.

</pre_check>

<context>

## Repository Structure

Full directory tree and "What Goes Where" table: [`.opencode/reference/structure.md`](.opencode/reference/structure.md).

Top-level directories: `journal/`, `projects/`, `areas/`, `resources/`, `archive/`, `inbox/`, `context/`, `codebases/`, `.opencode/`.

Workflow prompts live in `.opencode/workflows/` (24 files). Commands live in `.opencode/commands/`. Sub-agent definitions live in `.opencode/agents/`. Reference files (conventions, structure, security rules) live in `.opencode/reference/`.

## Automation Surface

### Slash Commands

- **Lifecycle workflows** — `/do <phase>` dispatches to `.opencode/workflows/`: `scout`, `research`, `brainstorm`, `plan`, `design`, `write`, `implement`, `test`, `self-review`, `resolve`, `debug`, `peer-review`
- **Knowledge graph workflows** — `/r <operation>` dispatches to `.opencode/workflows/`: `enrich`, `sync`, `plan`, `discover`, `ops`, `ingest`
- **Daily workflows** — direct dispatch: `/morning`, `/evening`, `/standup`, `/weekly`, `/capture`, `/meeting`
- **Utility commands** — `/bootstrap`

### QMD — Semantic Search

- Index: `node .opencode/scripts/qmd-index.js` (`--changed` for fast early-exit, `--full` to force re-embed)
- Query: `qmd query "<question>"`
- Re-index after any bulk create/actualize, `/r ingest`, or `/r sync` run.
- Collections are registered in `~/.cache/qmd/index.sqlite` via `qmd collection add`. The `/bootstrap` command registers all standard collections on first setup.

</context>

<trigger_table>

| Condition | Action |
|---|---|
| New session started | Execute Tier 0 load |
| Task touches writing, resources, people, teams, or projects | Execute Tier 1 load |
| Task involves coding in external repo | Execute Tier 2 load |
| User shares new project/role/team/tool fact | Update `context/context.md` or `codebases/<name>.md` immediately |
| Person, team, process, or concept referenced without resource article | Create stub article immediately |
| Learn new build command, architecture detail, convention, CI step, or decision while coding | Update `codebases/<name>.md` immediately (NEVER defer) |
| `context/context.md` empty or missing | Direct user to run `/bootstrap` before proceeding |

</trigger_table>

<steps>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [retran/meowary](https://github.com/retran/meowary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

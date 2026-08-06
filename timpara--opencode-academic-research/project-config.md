---
trigger: always_on
description: These rules tell OpenCode how to behave when working inside this repo.
---

# Project agent rules — opencode-academic-research

These rules tell OpenCode how to behave when working inside this repo.

## What this repo is

This is the OpenCode port of [academic-research-skills](https://github.com/timpara/academic-research-skills), itself a fork of the upstream Claude Code plugin by Cheng-I Wu (Imbad0202). It provides four skills, sixteen slash commands, and four custom subagents for academic research, paper writing, peer review, and revision.

The repo ships:

- `skills/` — four self-contained skill bundles auto-discovered by OpenCode
- `commands/` — sixteen `/ars-*` slash commands
- `plugins/` — one TypeScript plugin (session announce, write-scope guard, compaction hook, shell env injection)
- `.opencode/agents/` — four custom subagents (`ars-researcher`, `ars-writer`, `ars-reviewer`, `ars-verifier`)
- `scripts/` — the upstream Python verification suite (~47k lines) for citation integrity, claim-faithfulness audits, and temporal verification
- `agents/`, `shared/` — agent definitions and reference material the skills load on demand

## Subagent routing

Commands route to specialized subagents to keep context focused:

| Agent | Role | Commands routed here |
|---|---|---|
| `ars-researcher` | Literature search, synthesis, systematic review | `/ars-lit-review`, `/ars-3w` (subtask) |
| `ars-writer` | Paper drafting, revision, format conversion | `/ars-disclosure` (subtask), `/ars-format-convert` (subtask) |
| `ars-reviewer` | Peer review simulation | `/ars-reviewer` |
| `ars-verifier` | Citation checks, cache ops, mark-read signals | `/ars-citation-check`, `/ars-cache-invalidate`, `/ars-mark-read`, `/ars-unmark-read` (all subtask) |
| `build` (default) | Full pipeline, planning, revision coaching | `/ars-full`, `/ars-plan`, `/ars-revision`, `/ars-revision-coach`, `/ars-abstract`, `/ars-outline`, `/ars-rebuttal-audit` |

Commands marked "subtask" run in a child session and return results without polluting the primary context.

## Working in this repo

- **Python**: use `uv` for everything. `uv run pytest scripts/` for tests, `uv run ruff check scripts/` for lint.
- **Skill edits**: every change to a `skills/*/SKILL.md` file must preserve the YAML frontmatter (`name`, `description`, `metadata.version`) — they drive auto-discovery.
- **Command edits**: every file in `commands/` must keep `description` and `compatibility: opencode` in frontmatter. Route to the appropriate `agent` (`build`, `ars-researcher`, `ars-writer`, `ars-reviewer`, or `ars-verifier`). Add `subtask: true` for lightweight commands.
- **Plugin edits**: `plugins/*.ts` files import from `@opencode-ai/plugin`. Run `bun install` in `.opencode/` or the repo root after pulling.
- **Subagent edits**: `.opencode/agents/*.md` files define subagent behavior. Keep `mode: subagent` and appropriate permissions.
- **Docs edits**: when you change anything users read (README, SETUP, QUICKSTART), the change must work with OpenCode invocation paths. Do not reintroduce Claude Code `/plugin marketplace add` examples.

## Routing discipline

The skills assume routing has already settled before they take over. If a user message could match more than one skill (research vs. write vs. review), ask a clarifying question with concrete options rather than guessing. See `shared/references/intent_clarification_protocol.md` for the upstream routing protocol — the same rules apply here.

### Routing precedence (runs before skill selection)

**Step 0 — Escape hatch:** if the user's first message begins with `[direct-mode]` (case-insensitive, leading whitespace stripped), record the fact, strip the prefix, and route directly to the named skill without cross-phase clarification.

Otherwise:

1. **Explicit clear intent** — user invokes a specific skill via `/ars-*` slash command, or uses an unambiguous trigger keyword that maps to a single skill (e.g., "lit-review this", "review my paper", "draft an abstract") → Route directly, no clarification.

2. **Cross-phase materials detected** — user provides artifacts spanning ≥ 2 pipeline phases without naming a specific skill (e.g., pre-written abstract + pre-collected literature; full draft + reviewer comments + bibliography) → **Clarify**. Do NOT auto-route to a single-phase agent. List candidate workflows as a-d options in the message body. See `shared/references/intent_clarification_protocol.md` for the template.

3. **Ambiguous intent, no materials** — user provides no artifacts and no clear request → Clarify.

**Anti-pattern (caused upstream #133):** Receiving ambiguous cross-phase materials and silently auto-routing to a single-phase agent based on which phase the materials "look closest to." This bypasses orchestrator-level reconciliation and lets the subagent inherit the full ambiguity without independent oversight.

### Skill routing rules

1. **academic-pipeline vs individual skills**: academic-pipeline = full pipeline orchestrator (research → write → integrity → review → revise → final integrity → finalize). If the user only needs a single function, trigger the corresponding skill directly without the pipeline.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timpara/opencode-academic-research](https://github.com/timpara/opencode-academic-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->

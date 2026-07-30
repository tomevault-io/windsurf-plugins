---
trigger: always_on
description: The single standard every execution group applies. Curate this file into every worker's prompt. Derived from the `skill-management` skill (Fable 5/Mythos 5 tuning): newer Claude models handle ambiguity, long runs, and verification well — skills must be shorter, outcome-driven, and less prescriptive.
---

# Fable 5 Skill Conventions — skills-fable5-revamp

The single standard every execution group applies. Curate this file into every worker's prompt. Derived from the `skill-management` skill (Fable 5/Mythos 5 tuning): newer Claude models handle ambiguity, long runs, and verification well — skills must be shorter, outcome-driven, and less prescriptive.

## Structural rules

1. `SKILL.md` starts with `---` at byte 0; frontmatter has `name` + `description`; `name` matches the directory.
2. Description is the retrieval hook, not the manual: one or two sentences — when to load + what behavior changes. No feature lists.
3. Command files (omni `commands/*.md`) keep their existing frontmatter shape (`description`, `arguments`) — content-only rewrite.
4. **No renames, moves, or deletions of existing files.** New sibling files under the skill dir (`references/`, `prompts/`, `templates/`) are allowed and encouraged.

## Budget rules (testable)

| Surface | Ceiling |
|---------|---------|
| `SKILL.md` | ≤ 200 lines (aim ~120) |
| `commands/*.md` | ≤ 40 lines |
| `agents/*.md` | ≤ 40 lines |
| `rules/*.md` | ≤ 30 lines |

- Bulk content moves to sibling files loaded on demand: embedded system prompts → `prompts/<name>.md` (the skill instructs Claude to Read it at dispatch time); catalogs/API dumps/long examples → `references/<topic>.md`.
- Every fact gets ONE canonical home. Other files link to it ("see `references/x.md`", "see the `omni-ops` skill § Routes") instead of restating it.
- Exceeding a ceiling requires a one-line justification in the group report.
- Ceilings are not targets: most files should land near the ~120-line aim. Summing every ceiling (≈ 3,568) does NOT meet the wish's ≤ 3,300 repo-total — the total is the binding constraint.

## Fable 5 behavioral clauses

Where a skill orchestrates work, adapt these clauses into it (adapt wording to the skill's voice; do not paste verbatim into all files):

- **Act on enough info** (interactive skills — brainstorm, wizard, pm): "When you have enough information to act, act. Do not re-derive settled facts or re-litigate decisions the user already made. Recommend one path and proceed when it follows from the request."
- **Tight scope** (fix, work, refine): "Do the simplest thing that satisfies the request. No unrequested features, refactors, abstractions, or compat shims."
- **Grounded progress** (work, dream, fix, report, docs — anything that dispatches or reports): "Before reporting progress, audit each claim against tool output from this session. Say exactly what is verified, what failed, what was skipped. Never present intentions as completed work."
- **Real checkpoints only** (all): pause only for destructive/irreversible actions, genuine scope changes, credentials, or ambiguity that changes the safe action. Delete enumerated pause-condition lists.
- **Assessment vs action** (trace, review, report): when the deliverable is findings, report and stop — no unrequested fixes.
- **Deliberate parallelism** (work, dream, council, pm): delegate only independent subtasks; give each subagent explicit context, expected evidence, and stop conditions; verify side effects before reporting success.
- **Outcome-first final message** (all): lead with what happened, then evidence, then next action. Complete sentences; no arrow-chains or private shorthand.

## Delete on sight

- Step-by-step narration of behavior Fable 5 does unprompted (how to read files, how to ask questions, generic "be careful" advice).
- Duplicated CLI reference already canonical elsewhere (link instead).
- Stale content: session-specific examples, dead flags, old model names, motivational filler.
- Exhaustive option surveys and forced question rituals before every action.
- ANY reasoning-extraction language ("show your chain of thought", "write out your thinking", "transcribe reasoning"). Replace with: "summarize the decision and evidence", "report the checks performed and their results".

## Current CLI reality (verified 2026-07-04 — re-ground everything in this)

The skills were written for the pre-v5 daemon/team CLI. That surface is DEAD. Baseline: `bun run skills:lint` exits 1 with **118 missing-command references across 13 skill files** (`genie agent` ×37, `genie team` ×30, `genie wish` ×11, `genie events` ×10, `genie project`/`metrics`/`spawn`/`sessions`/`send`/`chat`/`broadcast`/`dir`, plus dead `task` subcommands).

- **Live genie v5 surface** (`genie --help`): `board`, `doctor`, `hook`, `init`, `install` (recreated by G8 as the install.sh finishing step), `launch <slug>`, `mcp`, `omni`, `setup`, `shortcuts`, `task`, `uninstall`, `update`. Task namespace: `checkout`, `create`, `done`, `export`, `list`, `status`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [automagik-dev/genie](https://github.com/automagik-dev/genie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

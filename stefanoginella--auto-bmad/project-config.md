---
trigger: always_on
description: This repo is a **BMAD standalone module** (one skill + a Claude `marketplace.json`). The skill
---

# CLAUDE.md — working in the auto-bmad repo

This repo is a **BMAD standalone module** (one skill + a Claude `marketplace.json`). The skill
(`auto-bmad`) is an orchestrator that runs the full BMAD story workflow one story at a time, on
**Claude Code or Codex**. This file is guidance for working **on the module**, not for using it.

## Core principle (do not violate)
The orchestrator **delegates BMAD work and reports** — it must never implement story work or run
`/bmad-*` skills directly. Every BMAD step (create-story, dev-story, code-review, TEA, retro,
project-context bootstrap/refresh) runs in a delegated `ab-*` sub-agent. **Preserve this separation
when editing.**

The orchestrator owns a small set of actions **directly** (never delegated) — all git/finalize
bookkeeping it already holds full pipeline context for. Don't "fix" these into delegated steps:
- **Git/PR work** — preflight, branching, per-phase commits, push, PR, and the Phase 9 merge prompt.
- **Phase 0 project-context probe** — an existence check that only decides whether Phase 2's
  bootstrap sub-step runs (the bootstrap itself is delegated like every other skill call).
- **Phase 9 finalize writes** — BMAD-status flip to `done` + the pre-push pipeline-report commit.
- **Phase 8 deferred-work archive** — at epic-end, after the project-context refresh, move
  fully-resolved entries out of the active `<impl>/deferred-work.md` ledger into the sibling
  `deferred-work-resolved.md` archive. No `/bmad-*` skill prunes the ledger, and the orchestrator
  already writes this file directly at Phase 7 — so this is connective bookkeeping, not a delegate.
- **Phase 7 HITL-halt feedback** — at the end-of-loop human halt, when the user added
  external-review changes, the orchestrator reads that diff directly to give a brief feedback
  summary before continuing (a lightweight read, **not** a delegated review — the one place it
  inspects code under any tier) and commits it like any other phase.

The mechanics of these live in the reference docs — **don't restate them here**: `git-and-pr.md`
(branching, push, PR, merge prompt) and `pipeline.md` (Phase 0 probe, Phase 7 HITL-halt feedback,
Phase 8 deferred-work archive, Phase 9 status flip + report commit). The only other time the
orchestrator does delegated step work itself is the `inline` delegation tier (see
`delegation-runtime.md`), and even then it follows the same phase contract.

## Delegation is tiered (the heart of the module)
BMAD abstracts neither sub-agent delegation nor per-agent model/effort, so we supply those with
tool-native files and degrade gracefully:
- **Tier 1 `custom-subagents`** (Claude Code, Codex) — each step runs in an isolated delegate at the
  profile's tuned model + effort (Claude `.claude/agents/ab-*.md`; Codex `.codex/agents/ab-*.toml`).
- **Tier 2 `general-subagents`** — generic subagents, no effort knob (effort not honored).
- **Tier 3 `inline`** — no subagents; run the step in-context (documented last resort).

`assets/agents/profiles.yaml` is the **single source of truth** (per-profile, per-tool model+effort
plus tool-neutral persona strings); `phase_profiles` maps each phase to a profile; and
`scripts/render-agents.py` generates the tool-native files from it. Host/mode are `auto` and
re-detected every run, so one provisioned project runs under either tool with no reconfiguration;
`target_tools` only controls which agent files get generated. Full detail: `delegation-runtime.md`
(host detection + the tiers) and `state-and-resume.md` (config/profiles schema, first-run).

## Layout & where behavior lives
- `.claude-plugin/marketplace.json` — Claude distribution (lists the single `./auto-bmad` skill).
- `auto-bmad/SKILL.md` — orchestrator entry point (On-activation gate + procedure). Keep it thin.
- `auto-bmad/references/` — where the real detail lives; each file owns one area:
  - `pipeline.md` — per-phase playbook.
  - `delegation.md` — exact per-skill prompts (tool-agnostic).
  - `delegation-runtime.md` — host detection + the three spawn tiers.
  - `tea-policy.md` — TEA risk rubric / selection.
  - `git-and-pr.md` — branching, commits, push, PR, merge prompt.
  - `state-and-resume.md` — config/state schema, first-run, profiles.
  - `overrides.md` — invocation-override vocabulary.
- `auto-bmad/assets/agents/profiles.yaml` — the single per-profile source (model/effort + persona
  strings). `claude/agent.md.tmpl` + `codex/agent.toml.tmpl` — one shared body template per tool the
  renderer fills in, so the four `ab-*` personas can't drift between tools.
- `auto-bmad/assets/module.yaml` + `module-help.csv` + `module-setup.md` — module identity,
  capability registry, and the self-registration/provisioning flow.
- `auto-bmad/scripts/` — dependency-free helpers, each with a `--self-test` and a self-documenting
  docstring (read the script for exact behavior):
  - `story_plan.py` — sprint-status reader.
  - `state_plan.py` — auto-bmad `state/{key}.yaml` reader (resume detection).
  - `render-agents.py` — agent generator from `profiles.yaml`.
  - `config_plan.py` — detects and additively heals drift between the shipped `profiles.yaml` and a
    project's runtime `config.yaml`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stefanoginella/auto-bmad](https://github.com/stefanoginella/auto-bmad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->

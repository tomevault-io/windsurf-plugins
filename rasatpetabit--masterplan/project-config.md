---
trigger: always_on
description: <!-- agentic-dispatch:central-pointer v2 -->
---

# AGENTS.md — `masterplan`

<!-- agentic-dispatch:central-pointer v2 -->
## Central agent policy

Cross-repo AskUserQuestion/ask_user_question (AUQ), RTK, Serena, Hindsight,
context-mode, and subagent/model-dispatch policy is centralized in the
agent-dispatch repo. Read it via `agent-dispatch where` (repo root) or
`agent-dispatch digest` (live routing policy). Do not duplicate or override
that policy here.

## What this repo is

`masterplan` is a Claude Code (and Codex) plugin providing the `/masterplan`
command. It orchestrates a **brainstorm → plan → execute → finish** development
workflow on top of [`obra/superpowers`](https://github.com/obra/superpowers)
skills. This file is the canonical project doc (AGENTS.md-primary as of
2026-06-10; the former CLAUDE.md-primary exception is retired — `CLAUDE.md` is
now the standard thin Claude shim).

As of **v8**, masterplan is a real Node codebase, not a markdown monolith. The
deterministic decisions live in **`lib/*.mjs`** behind **`bin/masterplan.mjs`**
(invoked throughout as `mp`) — zero-LLM-token, unit-tested. The markdown prompt
is a thin **sequencer (~800 lines)** that only orders `mp` calls, agent
dispatches, and gates. Durable state lives in `docs/masterplan/<slug>/state.yml`.

It is built in **five layers**:

- **L0 — Run bundle.** `docs/masterplan/<slug>/` (`state.yml` is the CD-7
  source of truth; bundle also holds `spec.md`, `plan.md`, `plan.index.json`,
  `retro.md`, `events.jsonl`, `handoff.md`). Flat YAML, atomic `tmp`+rename.
- **L1 — Thin shell.** `commands/masterplan.md` (the sequencer prompt) +
  `bin/masterplan.mjs` (`mp`, fs-only subcommands) + `lib/*.mjs` (~20
  pure-logic modules — core: `resume.mjs`, `bundle.mjs`, `plan-merge.mjs`,
  `wave.mjs`, `routing.mjs`, `finish.mjs`; periphery: `worktree{,-fs}.mjs`,
  `owner{,-fs}.mjs` (Guard D), `github-coord.mjs`, `qctl-*.mjs`,
  `codex-host.mjs`, `review-companion.mjs`, `hygiene.mjs`, `migrate.mjs`,
  `paths.mjs`). **L1 is the SOLE durable state writer (CD-7); the shell owns
  git, `bin` is fs-only.**
- **L2 — Workflow engine.** `workflows/execute.workflow.js` (one wave per
  launch) + `workflows/plan.workflow.js` (parallel planning fan-out). Returns
  digests/fragments only — **never writes state or commits.**
- **L3 — Agents.** `agents/*.md` (`mp-spec-decomposer`, `mp-planner`,
  `mp-subsystem-planner`, `mp-implementer`, `mp-plan-reviewer`,
  `mp-adversarial-reviewer`, `mp-explorer`). Bounded briefs; no session history.
- **L4 — Doctor.** `bin/doctor.mjs` + `lib/doctor/*.mjs` modules. Each finding
  is `{id, severity ∈ PASS|WARN|ERROR|SKIP, summary, fix}`; non-zero exit iff
  any `ERROR`.

The rest of the package:

- `skills/masterplan/SKILL.md` — the Codex-visible entrypoint (loads
  `commands/masterplan.md` and adapts tool names)
- `skills/masterplan-detect/SKILL.md` — auto-suggests `/masterplan import`
  when legacy planning artifacts are found
- `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` — plugin
  manifest + marketplace catalog (`rasatpetabit/masterplan`)
- `.codex-plugin/plugin.json` — Codex plugin manifest for the same command surface

Codex can host the command through `/masterplan:masterplan`. When it does, `§0`
host-detect reports a Codex host (`isCodex`), which lacks Claude Code's Workflow
tool, so the orchestrator runs waves on the foreground-sequential path
(`mp continue --codex-suppressed`); persisted `codex.routing` / `codex.review`
are unaffected and still apply to Claude Code runs.

## Where to read first

| If you need... | Read |
|---|---|
| The orchestrator prompt itself (L1 — the sequencer) | [`commands/masterplan.md`](./commands/masterplan.md) |
| Deterministic logic (the real "source code") | `lib/*.mjs` behind `bin/masterplan.mjs` |
| Layer-by-layer internals + failure modes | [`docs/internals.md`](./docs/internals.md) index → `docs/internals/{bundle-resume,wave-dispatch,plan-parser,task-verification,doctor}.md` |
| Public-facing overview + install + usage | [`README.md`](./README.md) · [`docs/install.md`](./docs/install.md) · [`docs/verbs.md`](./docs/verbs.md) |
| Release history + decision rationale per version | [`CHANGELOG.md`](./CHANGELOG.md) |
| Cross-cutting rules (CD-1…CD-10) + plan-field contract | `docs/conventions/cd-rules.md` · `docs/conventions/plan-annotations.md` |
| Active plans (current work) | `docs/masterplan/*/state.yml` (source of truth per CD-7) |

**Canonical reading order for a new session:** this file →
`commands/masterplan.md` (the sequencer) → the relevant `lib/*.mjs` for the
decision you're touching → `docs/internals.md` for design context → any active
run state in `docs/masterplan/*/state.yml`.

## Top anti-patterns (don't do these)

1. **Don't run substantive work in the shell's own context.** Dispatch to
   agents (`agents/*.md` via the L2 engine), `mp` subcommands, or
   `superpowers` skills. The orchestrator context holds sequencing state only —
   never raw file contents or verification dumps. Model selection for
   dispatches follows the central routing policy (`agent-dispatch resolve`);
   never hardcode model tiers here.
2. **Don't end a turn with a free-text question.** Use
   `AskUserQuestion`/`ask_user_question` with 2–4 concrete options (CD-9).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rasatpetabit/masterplan](https://github.com/rasatpetabit/masterplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->

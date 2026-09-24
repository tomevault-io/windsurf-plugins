---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

**Every verb gains a next-move stderr hint (next-leg-hints).** After each
successful, non-exempt move, the CLI now prints one line to stderr —
`next: <recommended move>` — naming what to run next (a leg-ending verb like
`export` or `summary` names the next skill/command; everything else falls
through to `devague status` / `devague plan status`). `status` and `plan
status` are exempt, since reporting the next move is already their whole
purpose. Emission is centralized in `devague/cli/_hints.py`, called exactly
once from dispatch — no command module emits it itself — so stdout (including
every `--json` payload) is untouched. The default per-verb text is
overrideable via `[tool.devague]` in `pyproject.toml` (`hints = false` turns
hints off globally; `[tool.devague.hints]` replaces one verb's text) or the
`DEVAGUE_HINTS` environment variable (`off` / `0` / `false`; the environment
wins over `pyproject.toml`). `devague explain <move>` documents the override
for the leg-ending verbs (`export`, `deviate`, `evidence`, `delta`, `summary`,
`today`), and `devague learn`'s operating rules mention it.

History lives in `CHANGELOG.md`.

## Working-backwards method

The agent drives the **deterministic** CLI — no LLM calls inside the CLI
itself. The workflow:

1. `devague new "<announcement>"` — the announcement-first entry point. The
   canonical first question is *"What's the announcement?"* ("Pretend this
   shipped successfully — what would you announce to users, teammates, or
   yourself?"). Creates a Frame seeded with the announcement claim
   (auto-confirmed, since it comes from the user). `devague learn` documents the
   full ten-stage guided sequence plus the always-on **operating rules** (the
   anti-fabrication contract); the portable, agent-agnostic version of that
   contract lives in `docs/llm-guidance.md` (#19).
2. `devague capture --kind <kind> "<text>"` — add claims; LLM-proposed ones
   (`--origin llm`) land as `proposed` and require explicit user `confirm`.
   Correct a claim in place with `devague amend <cN> [--text] [--kind]
   [--reason]` (#84) — it keeps the id, so honesty conditions, hard questions,
   `instruction`, and inbound `scope --seeds` refs all stay pointed at
   something real; the superseded `(text, kind)` pair lands on
   `Claim.revisions` and a confirmed claim flips back to `proposed`.
3. `devague interrogate <claim-id>` — attach honesty conditions and hard
   questions; honesty conditions from the LLM are also `proposed`. A blocking
   hard question is closed out with `devague interrogate <cN> --resolve <qN>
   --decision "<text>"` (#48/#52) — a USER decision, the claim-level twin of
   `park --resolve`; without it a blocking question deadlocks `converge`
   forever.
4. `devague confirm <id>` / `reject` / `park` — **all honesty conditions
   routed through the user**; the agent must not auto-confirm LLM proposals.
   Rejecting a claim cascades onto its still-live honesty conditions and
   unresolved hard questions, echoing `(also rejected: h3, q1)` (#83).
5. `devague scope "<surface>" --finding "<text>" [--seeds <cN|qN> …]` — record
   pre-frame exploration as first-class provenance; `--seeds` takes claim ids
   *or* claim-attached hard-question ids (#84), and `scope --amend <sN>
   --finding` corrects a finding in place.
6. `devague converge` — evaluates the convergence gate; lists remaining gaps.
7. `devague export` — only succeeds after `converge` passes; writes a
   buildable spec-md to `docs/specs/`. Verbatim claim text is markdown-escaped
   at render time (`render/_md_safety.md_safe_text`, #87) — presentational
   only, the stored JSON is untouched. A confirmed claim named by an approved
   deviation's `--affects` renders a `contested by <dN>` marker (#92): the
   spec is never rewritten, it points forward to the deviation ledger.

Full design: `docs/superpowers/specs/2026-05-23-devague-working-backwards-design.md`.

## Spec→plan method (the forward leg)

The **plan engine** is the structural peer of the frame engine — same chassis,
same anti-fabrication rules, no LLM inside the CLI. It is namespaced under the
`devague plan` subcommand group (the *skill* is `/spec-to-plan`; the CLI verb is
`plan` — they intentionally differ, mirroring how `/think` drives the flat
verbs). The workflow:

1. `devague plan new --frame <slug>` — seed a plan from a **converged** frame.
   Derives **coverage targets** (the frame's confirmed claims + honesty
   conditions). Refuses an unconverged frame; refuses to clobber an existing plan.
2. `devague plan task "<summary>" [--accept … --dep … --covers … --origin]` —
   add tasks; `--origin llm` lands `proposed` (user must `confirm` — and
   `plan confirm` / `plan reject` take many ids in one transactional call, #86).
   Refine with `accept` / `depend` (or `depend --remove` to cut an edge, #68) /
   `cover` / `instruct` / `amend` (edit a task's summary and/or replace/remove
   acceptance criteria by index, #68). Amending or demoting a CONFIRMED task
   flips it back to `proposed` and echoes that flip to stdout (#67). `--dep` /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentculture/devague](https://github.com/agentculture/devague) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

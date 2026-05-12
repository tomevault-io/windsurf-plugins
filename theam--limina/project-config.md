---
trigger: always_on
description: Limina is a research-first contract for autonomous technical investigation.
---

# Limina — Shared Runtime Contract

Limina is a research-first contract for autonomous technical investigation.

This file is the shared machine-facing instruction surface. Keep it short, specific, and stable. Runtime-specific details belong in adapters, skills, or scoped rules.

## Core Rules

1. Durable state lives in `kb/`. If it only exists in chat, it is not persistent.
2. The only required core workflow is `H -> E -> F`.
3. Do not create an experiment before the hypothesis exists.
4. Do not create a finding before the experiment exists.
5. `ACTIVE.md` is the only always-on state file. Keep it narrow.
6. Keep files small and specific. Prefer a concrete artifact over a generic note.
7. Store reusable lessons in `kb/lessons/` as small topic files. Read only the ones you need.
8. Use `CR` and `SR` only for real review points: major criticism, reset, or direction change.
9. If the evaluation, baseline, or prior state looks untrustworthy, stop optimizing and resolve that first.
10. Run `python3 scripts/kb_validate.py` after substantial kb edits and before closing kb-heavy work.
11. Every note in the research core must include a `## Links` section with real wikilinks.
12. Artifact notes alias their ID in frontmatter, so use `[[H001]]`, `[[E003]]`, `[[F010]]`, and similar links instead of raw file paths. Fixed notes use `[[ACTIVE]]`, `[[CHALLENGE]]`, and `[[DASHBOARD]]`.

## Mission First

Research exists to improve the mission, not to produce activity.

- Optimize for decision-quality progress, not local artifact completion.
- Treat benchmark or eval wins as mission progress only when the underlying capability is credible.
- Prefer mechanisms that should generalize beyond the current slice, wording, customer, or benchmark.
- If the current eval, baseline, or assumptions are not trustworthy, fix that before optimizing further.
- If the problem changes, update the research graph to reflect the new reality instead of forcing continuity.

## Working With The User

The user is the source of challenge framing, domain context, priorities, constraints, access, and missing information.

- Ask early for missing data, access, tools, context, decisions, or clarifications that materially affect the research direction.
- Do not stay silently blocked behind weak assumptions. Surface the uncertainty before building on it.
- When asking, state what you need, why it matters now, and what trade-off or degraded path exists if you do not get it.
- Ask narrowly scoped questions that unblock the next decision, not broad open-ended interviews.

## Session Recovery

Use this at the start of a session, after compaction, or whenever the current context may be partial or stale.

1. Read `kb/mission/CHALLENGE.md`.
2. Read `kb/ACTIVE.md`.
3. Open only the linked artifacts needed to reconstruct the current working state. Do not browse unrelated notes until the active scope is clear.
4. Reconstruct the current state before acting: current objective, next step, blocker, latest decisive evidence, and open decision.
5. Cross-check key numbers, dates, baselines, and decisions across the artifacts you read. If they disagree, treat that inconsistency as a blocker until resolved.
6. Search `kb/` before creating a new hypothesis, finding, or review.
7. Only continue once the active scope and state are coherent.

## Default Research Loop

Use this loop unless the current task clearly requires something else:

1. Frame the current decision, bottleneck, or uncertainty.
2. Identify the highest-value unknown blocking progress.
3. Choose the primary skill for the current phase before doing non-trivial literature search, experiment design, adversarial review, or implementation work.
4. Search by mechanism and failure mode, not only by task label.
5. Create or revise `H` so it states a falsifiable claim, why it should work, why it might generalize, and what shortcut risks remain.
6. Design and run one decisive `E` with a fair baseline, explicit success criteria, and clear guardrails.
7. Write `F` as what the evidence actually established, what improved for real, and what debt remains.
8. Update `kb/ACTIVE.md` with the new objective, next step, blocker, and working set.
9. Escalate to `CR` or `SR` only if the direction, trust in the setup, or strategic framing has changed.

## Skill Routing

Skills are part of the default process, not optional extras. Before major work, identify the dominant bottleneck, pick the narrowest matching skill, and use that skill as the primary playbook for the phase.

- Use `$exploratory-sota-research` when the main gap is external landscape search, mechanism discovery, benchmark mapping, or reproducibility scanning. Do not run a substantial literature or SOTA sweep without it.
- Use `$experiment-rigor` when the main gap is hypothesis quality, experiment design, baseline fairness, metrics, guardrails, or interpreting whether a result is valid, invalid, or inconclusive. Do not design or interpret a non-trivial experiment cycle without it.
- Use `$research-devil-advocate` when the main gap is adversarial review of the current direction: continue, continue with fixes, pivot, stop, or escalate. Do not perform a serious direction review or reset without it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theam/limina](https://github.com/theam/limina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->

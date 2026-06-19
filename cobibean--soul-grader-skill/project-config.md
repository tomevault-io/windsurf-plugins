---
trigger: always_on
description: Use when grading, reviewing, rewriting, or approving a Hermes Agent SOUL.md. Uses the SOUL.md field-guide research artifacts as the only normative source for what makes a good SOUL.md.
---


# SOUL Grader

## Linked references

- `references/fleet-soul-grading-workflow.md` — fleet-wide grading workflow: active/retired classification, companion-doc contradiction checks, non-Hermes service handling, durable report shape, and secret-safe archive handling.
- `references/research-deliverable-and-fleet-remediation.md` — deep-research/swarm deliverable pattern, polished static HTML review surfaces, and live fleet SOUL remediation notes.

## Overview

Use this skill to grade a Hermes Agent `SOUL.md`, draft a SOUL review, or turn a weak SOUL into a stronger one. The grading standard is intentionally narrow: **use the linked SOUL.md research artifacts as the only normative source for what makes a good SOUL.md.**

Do not import generic prompt-engineering advice, personal taste, web articles, model-provider docs, or vibes into the grade. You may use tools to read the SOUL being graded and to verify Hermes runtime facts, but quality judgments must come from the research artifacts bundled with this skill.

## Public / SSR publication posture

This is an unofficial community skill for Hermes Agent. The bundled references are intended to be safe for SSR sharing and public release: examples are anonymized, private workspace paths are removed or made relative, and no secrets or live deployment facts should appear in the bundle. If you add new examples or evidence, keep the same standard: cite public Hermes docs/source paths or anonymized patterns, not private customer, user, account, host, or credential details.

## Required source files

Before grading, load at least the grading standard reference:

```text
skill_view(name="soul-grader", file_path="references/soul-md-grading-standard.md")
```

Use the other references when you need more detail or citations:

```text
skill_view(name="soul-grader", file_path="references/soul-md-field-guide.html")
skill_view(name="soul-grader", file_path="references/soul-md-wording-verbiage-layer.md")
```

Source hierarchy for grading:

1. `references/soul-md-grading-standard.md` — canonical grader rubric and procedure.
2. `references/soul-md-field-guide.html` — full research report and evidence ledger.
3. `references/soul-md-wording-verbiage-layer.md` — detailed wording, verbiage, weak/strong examples, and slop detector.

If these references conflict, use the higher-ranked source. If the references are unavailable, stop and report that the grader source bundle is missing instead of grading from memory.

## When to use

Use when the user asks to:

- grade, score, audit, review, or approve a `SOUL.md`
- compare two `SOUL.md` files
- rewrite or improve a `SOUL.md`
- check whether a new agent identity is ready to deploy
- create acceptance criteria for a SOUL file
- diagnose agent identity drift, overreach, genericness, or approval-boundary failures
- decide what belongs in `SOUL.md` versus `CLAUDE.md`, `AGENTS.md`, skills, memory, manifests, or operator guides

Do not use as the sole workflow for:

- installing or configuring Hermes itself — load `hermes-agent`
- full new-agent intake/design — load the project’s new-agent intake or design skill too, if one is available
- authoring a new skill — load `hermes-agent-skill-authoring` too
- editing a user’s actual SOUL file without permission

## Strict source rule

The target `SOUL.md` being graded is evidence, not a standard. Runtime/tool output is evidence about deployment state, not a standard. The bundled references are the standard.

Allowed sources:

- the user-provided SOUL text or file path
- adjacent files only when checking contradictions or placement, such as `CLAUDE.md`, `AGENTS.md`, manifests, roster entries, or operator guides
- live Hermes/runtime output only for runtime hygiene checks
- this skill’s linked references

Not allowed as grading sources:

- generic prompt-engineering heuristics not present in the references
- web search results
- model-provider documentation
- personal preference unless the user explicitly asks for a custom overlay after the source-grounded grade
- unstated assumptions about a deployed agent’s access, host, credentials, or service state

## Grading rubric

Score out of 100 using the reference-defined categories:

| Category | Points | What to evaluate |
|---|---:|---|
| Mission clarity | 15 | Names who/what the agent serves and what outcome matters. |
| Identity + negations | 12 | Says what the agent is and what it must not become. |
| Core thesis | 10 | States the durable decision lens about the user/domain/problem. |
| Optimization hierarchy | 10 | Ranks tradeoffs instead of listing virtues. |
| Hard constraints | 10 | Includes 3–5 true filters with approval/override semantics. |
| Soft preferences | 8 | Separates scoring signals from bans. |
| Authority + escalation | 10 | Allowed / ask-before / never boundaries are clear. |
| Voice + truthfulness | 10 | Covers tone, vocabulary, never-claims, and evidence thresholds. |
| Success / artifacts | 8 | Defines durable/verifiable completion. |
| Artifact separation | 5 | Keeps commands, workflows, secrets, and volatile state elsewhere. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cobibean/soul-grader-skill](https://github.com/cobibean/soul-grader-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

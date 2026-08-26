---
trigger: always_on
description: `AGENTS.md` is directory-scoped guidance, not a role selector. This root file
---

# Autonomous Workshop agent instructions

`AGENTS.md` is directory-scoped guidance, not a role selector. This root file
applies to any coding-agent session operating in the source repository. Shared
architecture rules come first. The section **Coding agents building this
repository** is specifically for agents modifying, reviewing, testing, or
documenting Workshop; it is not the product-run workflow.

A normal product run is launched in a separate persistent toy project. The host
materializes the complete `.agents/product-run/` template there, including its
root `AGENTS.md` and nested `.agents/skills/autonomous-workshop/SKILL.md`.

## Shared runtime architecture

Autonomous Workshop is a thin, trustworthy workflow harness around a native
coding-agent runtime. Codex is the implemented Manager runtime; Claude Code and
Grok Build are planned adapters to the same boundary. One product run gives the
selected runtime the cognitive and tool-using work. The Workshop host retains
lifecycle order, durable state, deterministic gates, budgets, and authorized
external effects.

The root native Codex session is the Workshop Manager. It may use Codex-native
subagents for bounded parallel or specialist work, including matching and
working as the selected Inventor. Those agents remain children of the one
product-run session; they are not Python workers or separately launched Codex
processes.

All implementation and product-run work must preserve these boundaries:

- `workshop wish` persists the exact Wish, creates a private run workspace, and
  launches one native coding-agent session before Match.
- `workshop resume` resumes that exact session id. Stages are durable lifecycle
  checkpoints, not separate one-shot model sessions or personas.
- Native Codex performs Match reasoning, research, concept exploration,
  creation, inspection, and repair with its own tools and applicable skills.
- The universal digital Playtest baseline is `agent-playtest`,
  `mechanical-check`, and `printability-check`, as returned by
  `ToyBlueprint.required_playtest_checks()`. Codex-authored assessment is not
  evidence of successful printing, physical fit, durability, or human response
  unless a host-replayed check or authenticated physical receipt proves it.
- Every active Match, Invent, Make, Playtest, or Release attempt uses
  one native Codex Goal with one objective, proof artifacts, and a verifiable
  stopping condition: the current stage finalizer succeeds. Only one Goal is
  active at a time. Codex works toward it by observing, acting, evaluating exact output,
  and improving. That loop is native-agent behavior, not a Python program.
  Wish and Deliver remain host boundaries rather than agent Goals.
- An Inventor is a declared specialist bundle. `TASTE.md` governs creative
  judgment; `inventor.json` identifies the specialist and binds its exact
  extension trees; the required `<id>-inventor` skill defines its
  primary method, while optional additional Inventor-prefixed skill trees may
  contain scripts, references, assets, and tested deterministic tools for
  specialist craft. Custom code may not become an agent scheduler,
  prompt loop, lifecycle engine, or effect path.
- The host materializes every eligible Inventor as an official project-scoped
  Codex custom agent under `.codex/agents/`, bound to its exact identity, Taste,
  and skill bytes. That directory is the sole Inventor roster in a run. Codex
  owns native spawning, routing, and synthesis. The root session alone receives
  host stage authority and submits a stage proposal; child agents cannot
  advance gates or perform external effects.
- Python is narrow trusted substrate: typed contracts, deterministic tools and
  gates, artifact hashing, checkpoints, exclusive run-mutation locks, budgets, sandbox/session
  boundaries, authorization, idempotency, receipts, and reconciliation.
- External-effect credentials never enter the native agent subprocess. The
  host alone performs authorized Factory, payment, manufacture, postage,
  carrier, or other authenticated effects.
- Model prose and self-scores are proposals. Only host-verified exact bytes,
  deterministic checks, and reconciled receipts advance a gate.

## Coding agents building this repository

This section is for agents building the Workshop itself. It does not tell the
per-Wish product-run agent how to Invent, Make, or Playtest a product.

Do not add a second Python agent framework. Python stage agents, structured
model calls, profile subprocesses, and Python-owned scoring or reward loops are
not extension points. Never add Python prompt chains, browsing strategy,
candidate fan-out, model judges, stage-role views, or repair reasoning.

Read `docs/NATIVE_AGENT_RUNTIME.md` and ADR 0012 before changing the CLI,
runtime, workflow, product-run instructions, or lifecycle orchestration. The
native-session path is the production architecture. Preserve useful
deterministic contracts and tests; do not reintroduce removed cognitive
orchestration as a compatibility layer.

## Repository ownership

- `src/cli/`: argument parsing, output formatting, and exit codes only.
- `src/workshop/runtime/`: native engine adapters and trusted state/effect
  boundaries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autonomous-ai/autonomous-workshop](https://github.com/autonomous-ai/autonomous-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->

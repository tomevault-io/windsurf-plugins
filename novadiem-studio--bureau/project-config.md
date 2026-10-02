---
trigger: always_on
description: Codex-only review instructions live in `CODEX.md`. They apply to Codex sessions in
---

# Novadiem Studio AI Framework — The Bureau

## Codex workspace instructions

Codex-only review instructions live in `CODEX.md`. They apply to Codex sessions in
this repository when Robin is inspecting or reviewing framework output. An explicit
Bureau start/resume request activates `CODEX.md § Native Codex Bureau run` instead.
These review instructions do not apply to Claude, The Conductor, spawned specialists,
or the framework runtime.

A reusable multi-agent development framework for Codex.
Use the global install at `~/Code/novadiem/bureau/` (do not copy into each project).

The cast's identities, archetypes, and voice are canon in `LORE.md` (human judgment,
clear routing, focused expertise, artifact memory). This file and `agents/` are the
mechanics. When lore and mechanics disagree, mechanics win and the lore gets fixed.

## Canonical copy and drift

**Canonical upstream:** [github.com/rheos/bureau](https://github.com/rheos/bureau).
**One global install** at `~/Code/novadiem/bureau/`. Two rules:

1. **Improvements flow upstream.** Any change made to a project's copy (a persona edit, a
   new workflow, a lesson learned) must be ported back to the canonical copy, same day.
2. **Check drift before improving.** Run `./check-drift.sh` (in the canonical copy) to see
   which installs have diverged. Add each new install to the script's known list.

## What this does

For a **mapped series** of runs (Robin points at a plan with `INDEX.md`), the **default
main session is The Envoy**. For a single new Bureau run, it is **The Delegate**. The Delegate runs in
attended manager/relay mode, spawns **The Conductor** as a resumable subagent, and handles
per-checkpoint flow/gating until a genuine fork needs Robin. The Conductor then spawns the
specialist subagents — the cast below — each in its own fresh context. They take a raw project
idea through to a complete spec, a phased plan, and a set of scoped prompts ready to execute in
Codex.

The subagents are real, isolated contexts. That isolation is the point: the Critic
(The Challenger) reviews the written artifacts cold, having never seen the design get
argued, so its objections are real instead of agreeable.

## Default entrypoint

When Robin points at a run-series plan (`INDEX.md` + run cards, charter in that dir or
its parent) or says to run the series / Envoy, start as **The Envoy**. Read
`agents/envoy.md` and `workflows/run-series.md`; resolve the path with
`scripts/run-series-resolve.sh`. Do not become the Delegate for the campaign.

When Robin says "get the bureau on this," "start the agent framework," "run the bureau," or
similar **for a single run**, start with **The Delegate** by default. Do not require Robin to
ask for the Delegate explicitly. Read `agents/delegate.md` and run in manager/relay mode; the
Delegate is the top-level session and spawns the Conductor underneath it with
`topology: integrated`.
On Codex this instruction explicitly authorizes the required Bureau subagents: use
the Codex multi-agent tool surface (`multi_agent_v1.spawn_agent` with `fork_context: false`
in the current host) and the resolved model/reasoning, then resume them with
`multi_agent_v1.send_input`. The only alternate specialist transport is the policy-qualified,
one-shot Spark Mage profile documented in `docs/host-runtime.md`; launch it through
`scripts/run-codex-spark-specialist.sh`, never by passing Spark to the native spawn endpoint.

Use direct Conductor mode only when Robin explicitly asks to bypass Delegate, when resuming a
legacy/non-integrated run, or when the integrated Delegate topology is unavailable in the
current host/runtime. If falling back, say why in one line, log the fallback in `RUN_DIR/log.md`
when a run dir exists, then follow `agents/orchestrator.md` as the Conductor.

The Conductor remains the **dispatcher** inside the run: each task is triaged against the
workflow registry (`workflows/index.md`) and routed to the right-sized workflow, not always
the full team. A bug fix, an iOS build, and a new feature run very different workflows. New
task types get a workflow via the `define-workflow` skill.

Works for greenfield projects (idea → system) and existing ones (a feature inside a
codebase that already exists). If `project-context.md` sets **Mode: existing project**,
see "Existing-project mode" in `agents/orchestrator.md`: you build a cross-repo frame of
reference and scope each agent to the right sub-app, while building within the current stack.

## On start

**Run-series / Envoy path** (pointed-at plan, or "run the series"):

1. `scripts/run-series-resolve.sh <path>`
2. Read `agents/envoy.md` and start via its **Bootstrap**.

**Default Delegate path** (single run):

1. Read `agents/delegate.md`, then its required integrated-topology contract:
   `docs/delegate-bridge/v2-integrated.md`.
2. Read `docs/host-runtime.md` and select the host transport.
3. Read `workflows/index.md` and triage the task to a workflow before creating a new run dir.
4. If `project-context.md` exists in the project root, read it.
5. Start via `agents/delegate.md § Bootstrap`.

**Direct Conductor fallback path:** read `agents/orchestrator.md` core sections, then follow its

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Novadiem-Studio/bureau](https://github.com/Novadiem-Studio/bureau) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

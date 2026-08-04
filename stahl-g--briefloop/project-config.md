---
trigger: always_on
description: Multi-Agent Brief Workflow is a subagent-first briefing toolkit.
---

# AGENTS.md

## Purpose

Multi-Agent Brief Workflow is a subagent-first briefing toolkit.

Python CLI commands provide onboarding, workspace setup, runtime handoff, source tooling, validation, audit checks, and final rendering. The selected agent runtime coordinates the brief workflow through handoff artifacts and role-specific agents.

## Instruction Scope

This repository contains both development code and runtime agent contracts.

In repository development mode, files under `.agents/skills/`, `.agents/hermes-skills/`, `.claude/agents/`, `.codex/agents/`, and `.opencode/agents/` are source assets to inspect, edit, and test. Their role instructions become active only when the corresponding runtime explicitly invokes that role or skill.

Use `briefloop run --workspace <workspace> --runtime operator` to create a generic runtime handoff. The handoff artifact, not this repository manual, is the execution contract for a specific brief run.

For BriefLoop operator protocol, use `.agents/skills/briefloop/SKILL.md`.

## Environment Separation

Keep three instruction environments separate:

- Personal Codex environment: `~/.codex/AGENTS.md` is private user-level guidance. Do not copy its personal workflows, private paths, or local preferences into this public repository.
- Repository development environment: this `AGENTS.md` guides contributors and coding agents working on the BriefLoop source repo.
- End-user brief workspace environment: generated workspaces use `config.yaml`, `sources.yaml`, `user.md`, runtime handoff artifacts, and role skills. They must not depend on this repository `AGENTS.md` as their execution contract.

Do not treat repository development instructions as user-facing product behavior. If users need guidance, put it in README, docs, CLI help, generated handoff artifacts, or runtime skills.

## Context Mode

### Repository development mode

If the current directory contains `pyproject.toml`, `src/`, `tests/`, or `scripts/`, treat it as the source repository.

Use repository files for development, debugging, tests, generated configs, and documentation updates.

### Generated workspace mode

If the current directory contains `config.yaml`, `sources.yaml`, `user.md`, and `input/`, treat it as an end-user brief workspace.

Use workspace files as task context. Treat repository README, examples, agent configs, and docs as references, not source evidence.

## Standard User Path

For a real brief workspace:

```bash
briefloop onboard
briefloop init <workspace> --from-onboarding onboarding.json
briefloop run --workspace <workspace> --runtime operator
```

For demo exploration:

```bash
briefloop init <workspace> --demo
briefloop run --workspace <workspace> --runtime operator
```

`run` is the standard user-facing runtime handoff launcher. Generic CLI users must select one canonical runtime explicitly; dedicated adapters inject their fixed identity. The public CLI is `briefloop`; `multi-agent-brief` remains a compatibility entrypoint with identical behavior for existing scripts and installs.

## Runtime Handoff

Supported runtimes:

- `hermes`: Hermes parent agent uses `delegate_task` children.
- `claude`: Claude Code uses the repository command workflow.
- `opencode`: OpenCode uses the repository command and agent files.
- `codex`: Codex uses repository agent instructions and generated configs.
- `operator`: host-agnostic workflow for environments without a dedicated adapter. Historical `manual` manifests are read-only until an explicit reset records a canonical runtime.

Canonical workflow:

```text
doctor
→ source discovery when configured
→ input governance when available
→ scout
→ screener
→ claim-ledger
→ analyst
→ editor
→ auditor
→ finalize
```

## Development Standards

Use these standards for every repository change, especially before opening, updating, or merging a PR.

For architecture orientation before roadmap-driven work, read `docs/agent-dev-guide.zh-CN.md`, `docs/agent-dev-prompt.zh-CN.md`, `docs/architecture-status.md`, `docs/MIGRATION.md`, `docs/orchestrator-contracts.md`, and `docs/support-matrix.md`.

### Core Charters

For the full charter, read `docs/charter/README.md`. Short form:

1. Smart agents have no authority; authoritative actions are deterministic; approved effects leave records.
2. Enforce rules with schema, validators, gates, transactions, events, or tests when possible.
3. Each control-plane field has one authoritative writer: Python writes state; agents draft; humans approve.
4. Source plans, candidates, search summaries, and model summaries are discovery material, not evidence.
5. Frozen artifacts are append-only; post-freeze changes require a new revision, event, or contamination record.
6. Resolve conflicts by declared precedence, not model persuasion; prompts cannot skip control duties.
7. Close cross-cutting invariants structurally, not path by path: one invariant per merge over its whole lifecycle; state × path matrix before code; authority in one record that recomputation reads; shared fail-closed control-file loading; same-PR instruction sweep; fail-open gaps never deferred; a second same-shape review finding forces the structural fix, not another path patch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stahl-G/briefloop](https://github.com/Stahl-G/briefloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

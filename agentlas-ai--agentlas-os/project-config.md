---
trigger: always_on
description: <!-- AGENTLAS-INSTALL-ENTRY -->
---

# Agentlas Core Engine Meta-Agent Team

<!-- AGENTLAS-INSTALL-ENTRY -->
> **Asked to install this repo, not to work on it?** This file is the
> contributor constitution and will not help you. Read the install block at the
> top of [README.md](README.md): it points at
> `scripts/install-all-runtimes.sh`, which writes only under `~/.agentlas`,
> `~/.local/bin`, and this host's own plugin/command-adapter directories.


## Repository Constitution: Local Main Only

This repository uses one canonical development line: the local `main` branch
in the canonical Agentlas OS checkout.

- Make every source change and commit directly on local `main`.
- Do not create feature, release, backup, agent-named, or temporary branches.
- Do not create Git worktrees. Use an external recovery directory or a verified
  Git bundle when a safety snapshot is required.
- Before editing, confirm the canonical checkout is on `main`, inspect status,
  fetch remote refs, and inspect `main...origin/main`. A GUI "Pull origin"
  button is not permission to pull or merge blindly.
- If local `main` is dirty, preserve it with a reviewed checkpoint commit before
  reconciling remote changes on that same branch.
- Push only `main` and intentional release tags. Do not publish side branches.

## Public Release Allowlist (Hard Rule)

Before any commit, push, tag, release, or upload to a public GitHub repository,
construct and review an explicit allowlist. Public source and release artifacts
may contain only files required for an end user to install and run the product,
plus public-facing `README`, `LICENSE`, and `CHANGELOG` material.

Never publish internal design or research documents, plans, benchmarks,
benchmark prompts or results, tests, fixtures, test data, scores, logs,
screenshots, signing material, certificates, credentials, environment files,
operator notes, private paths, local memory, or recovery artifacts. Run tests
and benchmarks only in local/private temporary storage or private CI; publish
neither their inputs nor their artifacts. Existing unrelated local work must
remain unstaged.

For a public release, do not use a blanket `git add -A`. Stage the allowlist
explicitly, inspect `git diff --cached --name-only` and the staged archive
manifest, scan the staged content for secrets/private paths, and stop the
release if any excluded class appears. A passing test does not authorize
publishing the test.

This repository is a portable four-agent meta-agent team. Use it to create or
package Agentlas-compatible single agents and multi-agent teams for Codex,
Claude Code, Gemini CLI, Antigravity, Cursor, OpenCode, OpenClaw, Hermes
Agent, Ollama-served local models (Gemma, DeepSeek — see
`docs/local-models.md`), and `AGENTS.md`-compatible runtimes.

## Source Of Truth

- Canonical entry point: `AGENTS.md`.
- Architecture ownership rule: `docs/source-of-truth.md`.
- Runtime split and sync boundary: `docs/runtime-sync-boundaries.md`.
- Third-party plugin contribution boundary: `CONTRIBUTING.md` and
  `PLUGIN_CONTRIBUTIONS.md`.
- Global command contract: `docs/global-command-contract.md`.
- Production Ontology Runtime: `docs/ontology-runtime.md`, `ontology/`,
  `bin/ontology`, and `scripts/verify-ontology-runtime.sh`.
- Agentlas Cloud runtime contract: `docs/agentlas-cloud-runtime.md`,
  `agentlas_cloud/`, `schemas/agentlas-manifest.schema.json`, and
  `templates/agentlas.json.tpl`.
- Hephaestus Network 2.0 contract: `docs/hephaestus-network-2.0.md`,
  `docs/runtime-fallback-adapters.md`, `agentlas_cloud/networking/`,
  `schemas/routing-card.schema.json`, `.agentlas/routing-card.json`, and
  `scripts/verify-routing-cards.sh`.
- Stormbreaker robust execution contract: `docs/robustness-protocol.md`,
  `docs/robustness-eval.md`, and `schemas/robustness-eval-result.schema.json`.
- Canonical Stormbreaker Goal + UltraCode harness:
  `docs/stormbreaker-goal-ultracode-harness.md`,
  `agentlas_cloud/networking/stormbreaker_harness.py`, and
  `schemas/stormbreaker-goal-ultracode-harness.schema.json`.
- Builder quality gate: `contracts/builder-interview-research-gate.md`,
  `docs/builder-quality-research-basis.md`,
  `templates/builder-interview.md.tpl`, `templates/research-sources.md.tpl`,
  `templates/tool-selection.md.tpl`, `templates/domain-expert-synthesis.md.tpl`,
  `templates/prompt-performance-contract.md.tpl`, and
  `templates/capability-eval-plan.json.tpl`.
- Portable support contracts: `docs/mode-classifier.md`,
  `docs/clarify-question-loop.md`, `docs/agentlas-auto-activation.md`,
  `docs/local-credential-store.md`, and `docs/skill-lifecycle-promotion.md`.
- Agent experience and MCP contracts: `docs/agent-experience-assets.md`,
  `docs/mcp-build-resolution.md`, the matching `schemas/*.schema.json`
  files, `.agentlas/mcp-policy.json`, and
  `scripts/verify-experience-assets-contract.sh`.
- Team members: `agents/10-single-agent-builder/agent.md`,
  `agents/20-multi-agent-team-builder/agent.md`, and
  `agents/30-agentlas-packager/agent.md`, plus
  `agents/40-session-agent-builder/agent.md`.
- Mode contracts: `modes/single-agent-creator.md`, `modes/team-builder.md`,
  `modes/agentlas-packager.md`, and `modes/session-agent-builder.md`.
- Portable runtime core: `.agents/agentlas-core-engine-meta-agent/agent.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentlas-ai/Agentlas-OS](https://github.com/agentlas-ai/Agentlas-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

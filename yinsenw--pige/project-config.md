---
trigger: always_on
description: Status: Active repository instructions
---

# AI Agent Instructions

Status: Active repository instructions
Baseline date: 2026-07-09
Last reviewed: 2026-07-10

This repository is intended to be developed heavily with AI assistance. Future agents must treat the design documents as the product contract, not background reading.

## 0. Prime Product Directive: A Simple General Agent

Pige is a simple general Agent strengthened by local knowledge. **Input once, knowledge
grows naturally:** Pi performs validated recoverable work; users inspect or undo, not approve each step.

When implementation exposes powerful catalogs, provider ecosystems, package ecosystems, model capabilities, permissions, indexes, local tools, or Agent internals, hide that complexity behind sensible defaults, progressive disclosure, or internal metadata. Do not turn upstream complexity into visible product complexity unless a user-facing workflow truly needs it.

For model setup, connect one service and choose one default model. One disclosure grants
routine bounded calls to that exact Profile; show quiet status, not repeat prompts.
Local-first means local ownership/truth and no Pige telemetry, not confirmation-first UX.
Sensitive/restricted content and endpoint drift keep their narrow gates. Hide routing,
matrices, marketplaces, and taxonomy until a tested runtime needs them.

### 0.1 Named Agent Roles

Per `docs/AI_DEVELOPMENT_GUIDE.md`, Project Management owns delivery; Product Planning,
contracts; UI Design, visual guidance; Development, code/evidence. Role crossing requires
delegation, and design sync precedes closure.

## 1. First Reading Order

For any non-trivial task, start small and expand only when needed:

1. `AGENTS.md` if it is not already loaded by the agent runtime.
2. `README.md`
3. `docs/START_HERE_FOR_AI_AGENTS.md`
4. The task-specific pack listed in `docs/START_HERE_FOR_AI_AGENTS.md`

Read `docs/VISION.md`, `docs/PRD.md`, `docs/TECH_ARCHITECTURE.md`, `docs/DATA_ARCHITECTURE.md`, or other large documents by relevant section for the current task. Do not load every design document by default; too much context reduces agent attention and increases the chance of mixing unrelated rules.

Use `rg` or the section map in `docs/START_HERE_FOR_AI_AGENTS.md` to find the exact sections needed. Load whole large documents only for broad architecture, product-scope, or data-ownership changes.

The task router owns all specialist reading packs, including security, privacy, support,
runtime-policy, retrieval, structured data, knowledge, recovery, settings, Pi, onboarding, and public
workflow triggers. Follow its matching row rather than maintaining another routing list
here. Historical research is never default evidence; consult it only when the router
names it for rationale or package curation.

## 2. Non-Negotiable Invariants

- Simplicity is a product invariant: default UI should minimize decisions, labels, modes, and visible technical metadata.
- Every text, follow-up, URL, or file enters one Pi Agent decision path. Host code may
  preserve attached evidence first and enforce safety, but must not use heuristics or a
  fixed format workflow to choose semantic intent. Retrieval, fetch, parsers, OCR,
  analysis, proposals, and writers are typed tools selected by Pi.
- Ordinary conversation works without local evidence. When personal knowledge is
  relevant, Pi prefers bounded local retrieval and cites what it uses; retrieval is an
  Agent-selected advantage, not a mandatory gate for every answer.
- Documentation simplicity is also an engineering invariant: add a new design document only when an existing owner document cannot hold the decision cleanly; prefer task-specific section reads over loading the full documentation library.
- PRD P0 is the `v0.1 Public Alpha` release acceptance scope, not a single task scope. Implementation work must follow Milestones and the v0.1 Implementation Playbook phase boundaries unless those owner docs are deliberately updated.
- Open local files are durable knowledge truth: Markdown owns narrative knowledge and
  versioned Dataset Bundles own structured knowledge. Hidden indexes, caches, and
  machine-local databases remain rebuildable working layers.
- Local note storage location must be visible and controllable through Settings > Knowledge Base > Vault & Note Storage.
- Every user-visible setting must declare owner, scope, storage, backup behavior, permission requirement, and apply behavior.
- Agent-affecting settings must be compiled into typed Agent Runtime Policy Context and enforced by owning services; prompt text alone is never the enforcement layer.
- When local knowledge is relevant, Agent context assembly retrieves locally and
  packages only selected cited evidence; it never sends the whole vault or large source
  bodies by default. General answers need no fabricated vault evidence.
- Original files and source assets remain user-owned evidence; Pige may reference, link, or copy them according to explicit source storage strategy, but must not force migration.
- Data lifecycle is trash-first for durable vault data: Agent, Skill, package, cleanup, reset, cancellation, and compaction flows must not permanently delete durable knowledge, source evidence, memory, conversations, proposals, or operation records.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YinsenW/Pige](https://github.com/YinsenW/Pige) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->

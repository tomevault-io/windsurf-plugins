---
trigger: always_on
description: Create a multi-role Agentlas team package with orchestrator, PM Soul, Memory Curator, Policy Gate, eval, QA, handoffs, and runtime adapters.
---


# Multi Agent Team Builder

## Mission

Create an installable Agentlas team package. The output must behave like a
small operating system with orchestration, memory, policy, evaluation, and
runtime adapters.

## Use When

- The ownership-boundary classifier found two or more roles that independently
  own memory/context, tools/permissions, and success criteria.
- Those role outputs need routing, synthesis, review, or produces/consumes
  handoff through an orchestrator/HQ.
- The user asks for a team, company, firm, roster, departments, HQ, debate,
  parallel workers, review gates, or multi-role ownership and the ownership
  boundary is confirmed.
- The job needs routing, memory curation, PM continuity, policy approval, evals,
  or evidence gates across more than one role.

## Builder Interview and Research Gate

Before writing the team roster, run `contracts/builder-interview-research-gate.md`.
Do not jump from a rough idea to a generic HQ/worker list. Ask an 8-12 question
first batch and continue follow-ups until the team mission, owner, user, worker
boundaries, handoff artifacts, tools/plugins, memory policy, safety gates,
examples, and evaluation rubric are clear. If single vs multi is unclear, ask
the ownership-boundary question before generation; do not infer from the word
"team" alone.

Research the team's domain before writing role prompts. Use official or primary
docs, similar agent repositories or comparables, GitHub examples,
academic/professional theory, and plugin documentation for selected tools.
Every worker role must be justified by a real domain ownership boundary from
the interview or research. Record selected and rejected tools/plugins with
permission, secret, fallback, and smoke-test notes. Write
`docs/domain-expert-synthesis.md` before finalizing the roster so interview
answers, repo patterns, theory, and tool choices become concrete specialist
role behavior.

## System Agents - OS-Resident, Never Packaged (owner decision 2026-08-08)

Do NOT create `agents/10-pm-soul/`, `agents/20-memory-curator/`,
`agents/30-policy-gate/`, or `agents/40-eval-qa/` inside the team. These
roles are OS builtins now: every runtime seeds them
(`builtin-agentlas-pm-soul`, `-memory-curator`, `-task-bias`) and enforces
policy/judging at host chokepoints. The team package carries only their
OUTPUT files (`.agentlas/memory-map.json`, `.agentlas/memory-tickets.jsonl`),
never their bodies. Authoring a substitute body is a build defect —
`team_shape` marks leftover copies for stripping.

The one system file still copied verbatim:

- `system-agents/orchestrator-protocol.md` -> `docs/orchestrator-protocol.md`

Team-specific coordination rules that the old editable sections used to hold
go in the team's `agentlas.md` context section instead. policy-gate and
eval-qa remain delegation concepts: never add allow/deny or judging logic
anywhere in the package.

## Must Include

- Runtime instruction files must be written in English. This includes
  `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, worker `agent.md` files, skill
  instructions, workflow/command adapters, handoff contracts, return contracts,
  and operating docs. Translate Korean or other-language source material into
  English role behavior before writing the team. Localized public copy and
  trigger examples may use the target user language.
- Orchestrator/HQ inside the generated team. Its body is team-authored but
  MUST follow `system-agents/orchestrator-protocol.md` and state so in its
  header; copy the protocol file verbatim to `docs/orchestrator-protocol.md`.
- Memory Ticket handoff wiring to the OS-resident Memory Curator (workers emit
  `## Memory Events`; the runtime queues the ticket). No PM Soul / Memory
  Curator / Policy Gate / Eval QA member folders - see "System Agents -
  OS-Resident, Never Packaged" below.
- Worker roles with clear boundaries.
- Handoff brief and return contracts per the orchestrator protocol.
- `.agentlas/company-blueprint.json` with team topology.
- `docs/builder-interview.md`.
- `docs/research-sources.md`.
- `docs/tool-selection.md`.
- `docs/domain-expert-synthesis.md`.
- `docs/prompt-performance-contract.md`.
- `.agentlas/capability-eval-plan.json`.
- `.agentlas/memory-map.json`, `.agentlas/memory-tickets.jsonl`, and
  `.agentlas/vault-references.json`.
- `.agentlas/mcp-policy.json` with system-global-first catalog resolution,
  one-pass consent, per-requirement degradation, and no server command, args,
  endpoint, or credential value.
- Runtime adapters for requested targets.
- `.agentlas/global-commands.json`.
- One orchestrator/HQ global command that acts as the public entry point for
  the whole team across Claude Code, Codex, Gemini CLI, Antigravity, generic
  AGENTS.md, and terminal adapters.
- `scripts/verify-team-package.sh <package-root>` passes before final status is
  `completed`.

## Ontology-Backed Generation

When mode classification applies the `ontology-backed-agent` overlay
(`modes/ontology-backed-agent.md`), the generated team gains a shared
knowledge layer:

- Activate the ontology runtime at the team root: seed
  `.agentlas/ontology-sources.json` and `.agentlas/ontology-inbox/`, and wire

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentlas-ai/Agentlas-OS](https://github.com/agentlas-ai/Agentlas-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->

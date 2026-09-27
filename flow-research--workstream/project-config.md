---
trigger: always_on
description: This repository is for Workstream, source-agnostic governed contribution
---

# AGENTS.md

This repository is for Workstream, source-agnostic governed contribution
infrastructure for work performed by humans, AI agents, or both.

## Core Definition

Workstream turns project-defined tasks, immutable submissions, policy-governed
checks, and policy-governed acceptance into trusted `ContributionRecord` facts. Those
facts establish who completed what, under which locked rules, using which exact
artifact, and with what verified outcome. Applications and economic systems may
consume the facts; they do not control Workstream lifecycle truth.

Flow Identity is the current v0.1 external authentication provider, not the
definition or ownership boundary of Workstream.

## Working Rules

- Workstream is developing its first, unreleased v0.1. Do not introduce
  backward-compatibility layers, compatibility aliases, parallel old/new
  implementations, or “legacy/modern” variants merely to preserve earlier
  development code. When changing a module, replace the superseded implementation
  and update its affected callers, tests, schemas and current documentation
  together. Remove obsolete code and tests that exist only to preserve obsolete
  behavior; retain or replace tests protecting required behavior. Renaming
  duplicate implementations is not cleanup.

  Perform this cleanup within the current work’s affected scope, not as a
  repository-wide prerequisite. Trace shared consumers before deleting shared
  code; identify any remaining dependency explicitly without adding another
  compatibility path. Preserve authorization, locked lineage, atomicity and
  immutable evidence. Business policy/submission versions and required external
  protocol identifiers are not backward-compatibility implementations. Code
  cleanup does not authorize deleting retained data.
- Keep wording consistent with `README.md`, `docs/glossary.md`, and `docs/architecture_lockdown.md`.
- Keep pre-submission intake quality checks distinct from post-submission work
  evaluation. Intake failures prevent Submission creation; post-submit results
  supply evidence for policy-governed routing, not checker-owned acceptance.
  The locked ReviewPolicy requires human review by default; when false, passing
  required checks invokes the same authorized final-acceptance operation used
  by human `accept`. Never fabricate a Review or reviewer contribution.
  Do not describe all checking as
  deterministic or confuse setup-agent policy proposals with runtime evaluators.
  Model-based judges require supported registered implementations; do not claim
  them live merely because setup uses an agent.
- Use the simple engineering loop:
  `Intent -> Plan -> Bounded Change -> Tests -> Review -> PR -> Human Merge`.
- Inspect existing owners, call paths, policies, contracts and tests before
  designing a change. Prefer extending the existing operation over adding a
  parallel subsystem. Different triggers for the same business outcome normally
  share one operation and transaction, with explicit trigger provenance.
  Add an abstraction, policy, state or workflow only for a concrete requirement
  the existing design cannot safely meet; explain that gap in the change record.
  Keep control flow and dependencies explicit. Prefer small, cohesive functions
  and the fewest concepts needed for current behavior; do not add frameworks,
  generic helpers, configuration knobs or indirection for hypothetical future use.
  Make required behavior testable through stable owner interfaces and focused
  fixtures. Tests should protect outcomes and failure boundaries rather than
  mirror implementation details. Use broad fixtures or mock graphs only when
  the integration risk requires them.
  Simplicity must preserve authorization, locked lineage and atomicity.
- Keep the engineering loop separate from the Workstream product lifecycle. Workstream product review decisions remain `accept`, `needs_revision`, and `reject`; internal engineering reviewer findings are process evidence, not product decisions.
- Codex-discoverable repository skills live under `.agents/skills/`.
- Codex custom reviewer agents live under `.codex/agents/`.
- The lead uses the repository's Astra default; delegated work uses Sol with
  high reasoning. `.codex/config.toml` and the custom agent files own executable
  settings. Explicit user/session selections take precedence; do not silently
  escalate models or claim an existing session changed model after editing TOML.
- Durable engineering context uses the smallest applicable Commitrail record
  under `.commitrail/`. `.commitrail/INDEX.md` is durable navigation; GitHub
  open pull requests are the transient-work view. Review evidence is never an
  active-work queue or authorization source.
- `CONTRIBUTING.md` is the canonical human and agent entry path. GitHub
  permissions and branch protection govern contribution authority. Planning
  artifacts explain work; they do not authorize or block it.
- Distinct initiatives may proceed concurrently in separate branches or
  worktrees.
- Do not add Claude-specific files unless the user explicitly asks for cross-tool support.
- Do not use old names such as "task-production control plane" or "Garden roadmap".
- Spreadsheet exports live locally under ignored `sheets/`; do not commit them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flow-Research/workstream](https://github.com/Flow-Research/workstream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

---
trigger: always_on
description: Guidance for coding agents working in this repository. These instructions apply
---

# AGENTS.md

Guidance for coding agents working in this repository. These instructions apply
to the whole repository unless a more specific `AGENTS.md` exists below it.
Explicit user instructions take precedence over this guidance.

## Agent Workflow

- Work independently as the current agent. Do not spawn or delegate work to
  subagents.
- Before editing, run `git status --short`, confirm the current branch, and
  inspect existing diffs in the files you intend to change. Preserve the user's
  existing work; do not overwrite or revert unrelated files or changes.
- Use `rg` for search and focused reads before editing. Do not assume module
  boundaries from filenames alone.
- Before committing, review the final diff and stage only the files or hunks
  belonging to the requested task.
- At completion, briefly summarize the changes, the checks actually run and
  their results, and any checks not run or blocked. Never report an unrun check
  as passing. When committing, include the branch and commit ID in the summary.

## Project scope

KIP is the Knowledge Interaction Protocol: a cognitive state protocol for Agent
memory. The repository contains the KIP 2.0 normative draft, reference Brain
policies, grammars, schemas, conformance artifacts, bounded models, a TypeScript
language toolkit and a VS Code extension.

This repository does not implement a production Cognitive Nexus or the complete
Brain service. Those live in the sibling projects
[anda-db](https://github.com/ldclabs/anda-db) and
[anda-brain](https://github.com/ldclabs/anda-brain). Use their actual code when a
task needs implementation evidence; do not infer their capabilities from this
repository's specification or model tests. Change downstream repositories only
when they are included in the requested scope.

## Sources of truth and layout

- `SPECIFICATION.md`: normative Core and runtime semantics, including final
  belief, temporal succession, dependency validity and recording repair. Read its
  Status section for the draft-identity rule, the scope gate and the companion list.
- `KIP-2.0-Memory-Interface.md`: optional five-intent Agent-to-Brain binding.
- `brain/KIP-2.0-Validated-Learning.md` and `brain/KIP-2.0-Brain-Runtime.md`:
  normative optional companions for Skill trials/evaluations and for durable
  workers, leases and dispatch. `KIP-2.0-Cognitive-Consistency.md` is only a
  redirect table to where its former sections now live.
- `KIP-2.0-Capsule-Specification.md`,
  `KIP-2.0-Optional-Profiles-and-Migration.md` and `KIP-2.0-Invariants.md`:
  additional normative contracts and the stable invariant registry.
- `grammar/`, `schemas/`, `profiles/`: normative syntax, wire/artifact shapes,
  the memory package (`cognitive-memory@2.0.0`), the general domain package,
  the `kip:memory-default` policy artifact and the Memory Interface levels.
- `KIPSyntax.md`: informative model-facing syntax card. Its executable examples
  must agree with the grammar and toolkit.
- `brain/`, `SelfInstructions.md`, `SystemInstructions.md`: reference cognitive
  policies and role cards. Algorithms here do not override protocol requirements.
- `packages/kip-lang/`: lexer, parser, syntax AST, formatter, diagnostics,
  executable AST lowering, canonical JSON and host helpers. It does not execute
  KIP or supply an authorization boundary.
- `packages/vscode-kip/`: editor integration consuming `kip-lang`.
- `conformance/`: the executable engine suite, fixtures, portable vectors,
  reference models, adapter runners and digest tooling. See
  `conformance/README.md` before changing these.
- `formal/`: bounded verification models and reports with explicit proof limits.
- `KIP-2.0-Architecture.md`: informative rationale; normative contracts take
  precedence. Resolution/evidence reports describe their recorded revisions,
  not an evergreen certification of current code.
- `design/`: frozen pre-consolidation rationale. Do not maintain it as a second
  specification. New rationale belongs in the current specification/architecture.
- `v1/`: frozen historical protocol and integrations, outside the active pnpm
  workspace. `v2/` is a navigation stub; current v2 sources are at the root.
- `migration/`, `post/`, `diagrams/`: migration guidance, essays and supporting
  visuals. They do not supersede normative sources.

## Protocol invariants to preserve

- Proposition existence does not establish belief. Use final BELIEF status,
  including slot conflicts and dependency validity; missing evidence is not false.
- Meaning, epistemic confidence, mnemonic accessibility, trust and authority
  remain distinct. Recall is read-only and never implicitly reinforces memory.
- Engine-authenticated origin is protected. Attribution is not representation;
  cognitive content cannot grant permissions or upgrade its own authority.
- Actor correction, world change and recording/extraction repair have different
  histories and permissions. Preserve immutable source and assertion payloads.
- Dependency checks include relevant selection/absence dependencies, current
  authorization, temporal boundaries and lifecycle changes, not just record IDs.
- Task/context scope follows memory products without splitting canonical

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ldclabs/KIP](https://github.com/ldclabs/KIP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

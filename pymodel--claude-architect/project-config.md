---
trigger: always_on
description: Claude Architect is a cross-platform Claude Code plugin for verified delegation to Codex, OpenCode, Pi, and Pythinker. Claude authors a versioned Delegation Spec; the runtime launches an untrusted Producer in an isolated Git worktree, freezes its output as a Candidate Artifact, independently verifies it, and exposes the exact artifact for review and human-controlled integration.
---

# Agent Guide

## Project purpose

Claude Architect is a cross-platform Claude Code plugin for verified delegation to Codex, OpenCode, Pi, and Pythinker. Claude authors a versioned Delegation Spec; the runtime launches an untrusted Producer in an isolated Git worktree, freezes its output as a Candidate Artifact, independently verifies it, and exposes the exact artifact for review and human-controlled integration.

The repository also contains a strictly non-mutating advisor, cross-platform process supervision, crash recovery, bounded redacted logging, and Producer capability routing.

## Sources of truth

When sources disagree, use this descending precedence:

1. JSON schemas in `runtime/schemas/` and protocol constants in `src/protocol/`.
2. Runtime behavior in `src/`; generated packaged output lives in `runtime/`.
3. Contract and adversarial tests in `tests/`.
4. The delegation workflow in `skills/delegate/SKILL.md` and role definitions in `agents/`.
5. Plugin manifests in `.claude-plugin/` and user-facing documentation.

When prose conflicts with an executable contract, correct the prose or explicitly migrate the contract. Never leave them silently inconsistent.

## Mandatory operating rules

### Code discovery

This repository is indexed by the `codebase-memory-mcp` server as project `Users-panda-Projects-active-claude-architect`.

- For code exploration, use graph tools first: `search_graph` for symbols, `get_code_snippet` for exact source, `trace_path` for call chains, `search_code` for graph-assisted text search, and `get_architecture` for structure.
- If the server or a required graph tool is unavailable, state that limitation, then fall back to repository search and direct file reads. Do not continue silently.
- If the index is stale after substantial changes, run `index_repository` before relying on it.

### Planning and approval

Before making a non-trivial change:

- Read the relevant schema, implementation, tests, skill text, and recent Git history.
- Inspect `tasks/lessons.md` and any active `tasks/todo.md` when present.
- State the intended outcome, authorized scope, and verification method.
- Inspect Git status and preserve unrelated user changes; never assume a dirty worktree is disposable.
- Identify implications for macOS, Linux, Windows, and each affected Producer.

Before invoking **dynamic workflows**, **ultra code**, or any equivalent harness feature that immediately launches a large swarm of subagents, explain the tradeoffs and obtain the user's explicit approval. Do not infer approval from a general request to investigate or implement.

### Scope and engineering quality

- Make the smallest change that fully satisfies the request. Every changed line must trace to an acceptance criterion.
- Do not perform unrelated reformatting, renaming, refactoring, or cleanup.
- Fix every lint failure, test failure, and flaky test discovered during the work, even when pre-existing or unrelated to the initial request.
- If a required fix exceeds the authorized scope, stop and request expanded scope. Do not ignore the failure or report the task complete.
- Prefer quality, simplicity, robustness, scalability, and long-term maintainability over short-term development cost.
- **Never propose deferring an applicable issue or fix.** If a problem is real and a fix applies, do the fix now — even when it requires a redesign or significantly more work. "Defer", "follow-up later", "out of scope for now", and equivalents are prohibited recommendations; the only permitted alternative to fixing immediately is stopping to request expanded scope, then fixing.

## Non-negotiable trust invariants

- Every delegated implementation or repair attempt starts with fresh context in a fresh isolated worktree.
- Implementers cannot review, approve, or accept their own work.
- Independent reviewers evaluate frozen candidate bytes without sharing implementer context.
- Read-only roles cannot mutate files, Git state, processes, or external systems.
- Roles communicate through versioned specs, manifests, patches, findings, and other durable artifacts—not hidden conversational state.
- Verification is objective, recorded, and rerunnable; Producer claims are never evidence.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PyModel/claude-architect](https://github.com/PyModel/claude-architect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->

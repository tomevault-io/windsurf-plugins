---
trigger: always_on
description: Remnic is a multi-platform memory system. Keep these boundaries intact on every change:
---

# Remnic - Agent Guide

## Architecture Boundaries (Non-Negotiable)

Remnic is a multi-platform memory system. Keep these boundaries intact on every change:

1. `@remnic/core`, `@remnic/server`, and `@remnic/cli` own Remnic's core behavior.
   Core memory semantics, storage, retrieval, extraction, governance, and standalone operation must live there.
2. Core and standalone paths must not depend on OpenClaw, Hermes, or any future host.
   Host integrations may consume core. Core must not reach back into host SDKs, config shapes, or runtime lifecycles.
3. Platform-specific behavior belongs in platform adapters only.
   OpenClaw-specific code belongs in `packages/plugin-openclaw` plus the current root `src/` compatibility wiring that still hosts OpenClaw runtime entrypoints today. Hermes-specific code belongs in `packages/plugin-hermes`. Keep host logic thin and translation-focused.
4. Do not reinvent host-native features.
   If OpenClaw, Hermes, or another platform already provides a runtime capability, plugin hook, command surface, or extension primitive, use that real upstream contract instead of recreating a parallel Remnic abstraction.
5. Verify host behavior against current upstream source and docs before implementing it.
   Issue text, old local docs, or remembered APIs are not enough for host-facing work.

## Upstream References

Use these as the canonical starting points for adapter work:

- OpenClaw repository: <https://github.com/openclaw/openclaw>
- OpenClaw plugin docs: <https://github.com/openclaw/openclaw/tree/main/docs/plugins>
- OpenClaw SDK overview: <https://github.com/openclaw/openclaw/blob/main/docs/plugins/sdk-overview.md>
- OpenClaw SDK entrypoints: <https://github.com/openclaw/openclaw/blob/main/docs/plugins/sdk-entrypoints.md>
- Hermes Agent repository: <https://github.com/NousResearch/hermes-agent>
- Hermes Agent docs/site: <https://hermes-agent.nousresearch.com>

## Adapter Implementation Rules

- Start from the host's current upstream contracts, then adapt Remnic core into them.
- Reuse upstream platform primitives when they exist; only add Remnic-owned glue where the host does not already solve the problem.
- Keep standalone and shared-core behavior testable without booting OpenClaw, Hermes, or another host.
- If a change touches both core semantics and a host adapter, land the core contract first and make the adapter consume it second.

## Cleaner PR Workflow (Mandatory)

These rules are the default workflow for all agents and contributors.

1. Keep PR scope narrow.
   - One subsystem group per PR whenever possible.
   - If work spans multiple groups, split it before review. The default split for memory-heavy work is:
     - schema/surface contract changes
     - storage/serialization/cache changes
     - retrieval/planner/freshness behavior changes

2. Sync with `main` before the first serious review cycle.
   - Rebase or merge `main` before requesting AI review.
   - Do not let a PR drift for multiple review rounds and then merge `main` halfway through unless forced by a conflict.

3. Batch review fixes by subsystem.
   - Re-scan unresolved comments, fix the whole subsystem, run verification once, then push once.
   - Avoid serial micro-pushes that only expose the next adjacent invariant.

4. Run the local hardening gate before claiming review-clean.
   - Always run `npm run preflight:quick`.
   - If you touch `src/` or `packages/remnic-core/src/` `orchestrator.ts`, `storage.ts`, `intent.ts`, `memory-cache.ts`, `entity-retrieval.ts`, or `config.ts`, also run `npm run test:entity-hardening`.
   - If Cursor CLI is available, run `npm run review:cursor` before requesting external AI review.

5. Treat external AI review as stale unless it matches the current head.
   - Do not call a PR clean if the latest positive AI verdict targets an older commit.
   - A merge-ready PR needs green checks, zero unresolved review threads, and a fresh positive AI verdict on the current head.

Reference workflow:
`docs/ops/pr-review-hardening-playbook.md`

## Why Stateful PRs Churn (Read Before Touching Lifecycle Logic)

PRs in retrieval, session identity, compaction, cache, or reset/end-of-session code
often attract many review rounds for the same structural reason:

1. The subsystem is stateful across multiple entrypoints.
   - A local fix in one hook can break `before_reset`, `session_end`, compaction,
     sparse metadata handling, remembered bindings, provider rebinding, or restart recovery.
2. Reviewers probe different slices of the same state machine.
   - One reviewer may catch provider detection drift.
   - Another may catch lifecycle drain gaps.
   - Another may catch stale-cache or replay behavior.
   These are usually adjacent invariant misses, not unrelated bugs.
3. Comment-by-comment patching makes churn worse.
   - If you only fix the literal review comment, the next review round often finds
     the neighboring invariant you did not model yet.

Required response:

1. Stop and model the full contract first.
2. Write the scenario matrix before changing code.
3. Patch the subsystem coherently once.
4. Add tests for the failure class, not just the reported instance.
5. Run the hardening gate before asking for another review.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshuaswarren/remnic](https://github.com/joshuaswarren/remnic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

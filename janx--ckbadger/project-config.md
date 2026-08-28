---
trigger: always_on
description: Instructions for AI agents working on ckbadger - a CKB blockchain explorer.
---

# AGENTS.md

Instructions for AI agents working on ckbadger - a CKB blockchain explorer.

## Project Principles

- **CKB Native** - Make CKB concepts tangible instead of just-another-explorer. CKB chain data is the only source of truth, all other data are derived from it.
- **Local First** - Optimized for decentralized deployment on localhosts
- **Agent Friendly** - Prefer clear, automation-friendly structure and workflows

### Local First (Expanded)

- Local-first aligns with Web5 and Unix philosophy. Files and executable binaries are the foundation of composability, and ckbadger is designed around files and executable binaries.
- Local-first means ckbadger optimizes for writes (building data indexes), not reads (serving API and web page requests), unlike typical blockchain explorers. This enables extremely fast database sync, so local experiments remain cheap: if the DB is broken, rebuild it instead of protecting a 60-hour sync artifact. DB reads remain very fast, just not the top optimization target.

## Design Starting Point (MANDATORY)

- Documents under `docs/prompts/` capture the deep understanding and thinking principles of ckbadger.
- Treat `docs/prompts/` as the starting point for all design reasoning and architecture decisions.

## Agent Task Template (MANDATORY)

For any non-trivial task, use this structure in the final summary or PR description:

```md
## Goal

- What problem is being solved

## Principle Alignment

## Result

- Behavior change summary
- Re-sync required: yes/no
- What to do next
```

**Principle Sync Rule**: If principle wording changes, update both `README.md` and `CLAUDE.md` in the same commit.

## Coding Principles (MANDATORY)

- **Fail Fast, Fail Early** - Never hide invariant violations with silent fallbacks, lower-bound clamps, or default-zero repairs; fail immediately with actionable context
- **Refactor First When It Helps** - Before implementing new code, evaluate whether a focused refactor will reduce complexity or risk; if yes, refactor first and then implement.
- **Single Calculation Path for Read Data** - For any data that must be read/derived, keep exactly one computation path and make that single path correct.
- **No Fallback Calculation Chains** - Reject defensive multi-path computation such as "if path A is wrong, fallback to B, then fallback to C"; do not add path B/C, fix path A.
- **No Workaround Fixes for Bugs** - Do not ship bypasses, route detours, temporary guards, degraded-mode switches, or UX-level evasions as bug fixes. Identify and fix the upstream root cause in the owning computation/write path.
- Do not add silent guards to mask bad states on correctness-critical paths (for example `max(0)`, `saturating_sub`, `unwrap_or(0)`).
- If an invariant is violated, return/raise an error with enough context (block/tx/key/date) to locate the upstream bug quickly.

## Debug & Fix Principles (MANDATORY)

- **Trace Root Cause** - Do not stop at shallow/near-surface symptoms; track the true upstream root cause.
- **Fix Root Cause, Not With Fallbacks** - If you find some data incorrect, don't be satisfy, don't use recalculation code to correct it, instead you should check why it's incorrect in the first place, fix the bug there. Do not patch incorrect pre-computation with extra fallback paths; fix the original computation logic that produced the wrong state.

## DB Responsibility Boundary (MANDATORY)

- **Indexer owns all chain-store RocksDB writes**: any operation that creates/updates/deletes persistent domain- or append-only-store state must be executed by `ckbadger-indexer`. The one exception is the separate **network store**, whose sole writer is the opt-in `ckbadger-crawler` service (see Network store responsibility below).
- **API is read-only for RocksDB**: `ckbadger-api` must only read from store (secondary/open_secondary path) and must not write persistent state.
- If API needs missing derived data, API must trigger indexer to compute and write it, then wait/poll for result instead of writing DB directly.
- **Domain store responsibility**: domain store (`[store].domain_data_path`, 59 CFs) holds all mutable canonical/query state including activities, addr_txs, live/consumed cell markers, and all indexes. May perform create/update/delete as required by chain progression and reorg handling, but only via indexer.
- **Append-only store responsibility**: append-only store (`[store].append_only_data_path`, 1 CF: `CF_CELLS`) holds only immutable cell payloads, content-addressed by outpoint. Write-once, never updated or deleted.
- **Append-only correction policy**: if cell payload data in the append-only store is wrong, fix indexer logic and rebuild from genesis; do not patch cell data with in-place update/delete.
- **Cross-store cell reads**: live/consumed markers live in domain store; cell payloads live in append-only store. Reading a full cell requires both the domain and append-only stores.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [janx/ckbadger](https://github.com/janx/ckbadger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

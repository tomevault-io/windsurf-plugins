---
trigger: always_on
description: Applies to this repository and all descendants unless a nested `AGENTS.md` adds more specific instructions.
---

# AGENTS.md

## Scope

Applies to this repository and all descendants unless a nested `AGENTS.md` adds more specific instructions.

## Purpose

This repository owns Via source and runtime behavior. It does not own live or desired-state deployment.

## Read first

Before changing non-trivial runtime behavior, read the relevant Via guide and the nearest crate README or config.

## Important paths

- Runtime: `core/`
- Verifier: `via_verifier/`
- Prover / Indexer: `prover/`, `via_indexer/`
- Bitcoin: `core/lib/via_btc_client/`, `core/node/via_btc_*`, and verifier BTC paths
- DA / Reorg: `core/lib/via_da_clients/`, `core/node/via_main_node_reorg_detector/`, and `via_verifier/node/via_reorg_detector/`. (When touching one, check the other — see *Reuse and duplication discipline*.)

## Source-of-truth rules

- Source code describes behavior; it does not prove deployment.
- Prefer Via-specific `via_` modules when extending fork behavior.
- Follow the call graph into upstream (non-`via_*`) code when necessary, and explain why a Via extension was insufficient.
- For Bitcoin work, remember that txids are byte-reversed.

## Safety rules

- Never commit secrets or live credentials.
- Do not run migrations or deploy from this repo without explicit approval.
- Keep local agent scratch directories (`.gitnexus/`, `.agents/`, etc.) out of commits.

## Reuse and duplication discipline

Apply this section before writing implementation code. The PR template’s *Reuse & Duplication* and *Performance, Complexity, and Resource Impact* sections cannot be filled in honestly after the fact — they exist to surface the reasoning required here.

**Production code is permanent audit cost.** Every new function, detector, client, poller, watcher, or worker adds ongoing review and maintenance burden. Production LOC is measured as **net delta** (lines added minus lines removed, excluding comments, documentation, tests, and generated files). Before writing new code, identify the existing function or module that should own the behavior. If you cannot name it, you are likely introducing duplication.

**Do not tunnel on `via_*`.** `via_*` directories are not hard boundaries. Follow the call graph into upstream and non-`via_*` modules. Always check sibling implementations across node types (main-node vs. verifier, `core/` vs. `via_verifier/`) before adding new logic.

Detector, poller, watcher, and fetch/compare logic copied across main-node and verifier paths is the canonical anti-pattern this section exists to prevent (see PRs #355–#360).

**Mandatory pre-coding check.** Before creating a new file, struct, detector, poller, watcher, client, DAL method, parser, or conversion in a `via_*` path, complete the following:

1. Name the closest existing function or module you considered extending or replacing.
2. List the specific sibling paths you inspected (main-node ↔ verifier, `core/` ↔ `via_verifier/`). For reorg-related work this must include both `core/node/via_main_node_reorg_detector/` and `via_verifier/node/via_reorg_detector/`.
3. If you are not extracting shared logic, state the exact invariant, ownership boundary, or execution-context difference that prevents extraction.

If you cannot complete the steps above, stop and do the search before writing implementation code.

### Anti-pattern vs Required Pattern (example)

**Anti-pattern** (duplication of sibling logic):

- Similar detector / poller / watcher logic added in both a `core/node/` path and the corresponding `via_verifier/node/` path with only minor differences.
- No attempt to extract shared behavior.

**Required pattern**:

- Shared logic extracted to `core/lib/` (or the most appropriate shared crate).
- Only thin node-specific wrappers remain in `core/node/` and `via_verifier/node/`.

This pattern applies to any duplicated logic across main-node and verifier (not just reorg detectors).

### Maintaining supporting files

When new duplication patterns, high-risk areas, or sibling relationships are discovered, update the following files:

- `.github/sibling-paths.yml` — Add new main-node ↔ verifier path pairs.
- `.github/via-scopes.yml` — Add new `via-*` scopes when a recurring area of work appears.
- `.github/via-areas.yml` — Map new paths to the appropriate `Via-Area` value.

Also consider adding a small `AGENTS.md` file in the relevant high-risk directory if one does not already exist.

## Commit Message Convention

Use Conventional Commits. All Via-specific changes must use a `via-` scope.

**Format:**

```text
<type>(via-<area>): <imperative subject>
```

**Allowed types:** `feat`, `fix`, `perf`, `refactor`, `test`, `docs`, `chore`, `build`, `ci`, `revert`

**Examples:**

- `feat(via-reorg): detect deep reorgs from DA layer`
- `fix(via-btc): correct txid byte-reversal in mempool watcher`
- `refactor(via-da): extract shared inclusion proof parsing`
- `perf(via-verifier): batch L1 batch metadata loads`
- `fix(via-reorg): align reorg handling with new upstream batch finality rules`
- `chore(upstream-sync): merge ZKsync vX.Y.Z`

**Scopes:** See `.github/via-scopes.yml` for the current list of allowed scopes, including Via-specific `via-*` scopes and the non-Via `upstream-sync` scope.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vianetwork/via-core](https://github.com/vianetwork/via-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->

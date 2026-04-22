---
trigger: always_on
description: <!-- Source: .cursor/agents/code-reviewer.md -->
---

<!-- Source: .cursor/agents/code-reviewer.md -->
<!-- Last synced: 2026-03-13 -->

# Ledger Live Monorepo

This is the **ledger-live** monorepo — a pnpm + turborepo workspace containing:

- `apps/ledger-live-desktop` — Electron desktop wallet (React, TypeScript)
- `apps/ledger-live-mobile` — React Native mobile wallet (iOS/Android)
- `libs/` — shared libraries (`ledger-live-common`, `client-ids`, `coin-modules`, etc.)

## Code Review Philosophy

- Flag only **clear violations** of the rules described in these instructions.
- Prefer **quality over quantity** — a few high-confidence comments are more valuable than many speculative ones.
- Focus on bugs, security issues, architecture violations, and missing tests before style nits.

## Changeset Requirement

Every PR that changes user-facing behavior or library APIs must include a changeset (`pnpm changeset`). Flag PRs that add features or fix bugs without one.

## Privacy & Security — `@ledgerhq/client-ids`

Sensitive identifiers (DeviceId, UserId, DatadogId) must always use the `@ledgerhq/client-ids` library:

- **Never** use raw string IDs for devices, users, or analytics.
- **Always** use `DeviceId`, `UserId`, or `DatadogId` classes from `@ledgerhq/client-ids/ids`.
- ID values are only accessible through explicit export methods (e.g., `exportUserIdForSomething()`).
- Every export method must be allowlisted in `libs/client-ids/export-rules.json` with a justification.
- Export IDs only at system boundaries (API calls, persistence) — never in the middle of processing.
- `toString()` and `toJSON()` return `[DeviceId:REDACTED]` by default — this is by design.

## Dependency Review

When a PR adds or updates dependencies in any `package.json`:

- The dependency must be justified (not duplicating an existing capability).
- Peer dependency compatibility must be verified.

## Lockfile (pnpm-lock.yaml) Review

When a PR touches `pnpm-lock.yaml`, check that the diff is **scoped to what the PR author actually changed** (e.g. the packages they added/updated in `package.json`). Flag the PR if the lockfile diff:

- **Unrelated version bumps** — Resolutions or versions of packages that were not added, removed, or updated in this PR’s `package.json` changes.
- **Large accidental rewrites** — Big reformatting, reordering, or mass version changes that go beyond the intended dependency change (e.g. running `pnpm install` in a different environment or with different pnpm/node).
- **Unwanted bundle or duplicate entries** — New lockfile entries (packages or versions) that don’t correspond to the PR’s stated dependency changes, or duplicate/conflicting entries that suggest lockfile corruption or merge issues.

If in doubt, the lockfile diff should be explainable by the set of `package.json` edits in the same PR; otherwise ask the author to regenerate the lockfile from a clean `pnpm install` and ensure only the intended dependency changes remain.

## Coin-specific logic and families contract

**Do not add coin-specific branches in generic UI.** Flag PRs that introduce `if (family === "evm")` (or similar) or coin-specific hooks in shared screens/ViewModels. Coin-specific behavior must live in **families/** and be exposed through the **families contract**:

- **Desktop:** Optional slots on `LLDCoinFamily` in `apps/ledger-live-desktop/src/renderer/families/types.ts`; families implement in `families/<family>/`; generic code uses `getLLDCoinFamily(currency.family).SlotName` only (no family-name checks).
- **Mobile:** Same idea: optional slots or generated maps (e.g. `NoAssociatedAccounts`), implemented per family; generic code looks up by contract, not by `family === "…"`.

When a flow needs new coin-specific behaviour, the fix is to **extend the contract** (new optional slot) and implement it in the family folder, not to add branching in generic code. This keeps the codebase ready for modularisation and lazy loading. See `.cursor/rules/coin-families-contract.mdc` for the full rule and the Scan Device “no associated accounts” example.

If a PR changes the coin-framework interface (`libs/coin-framework/src/api/types.ts`), the author should also update the developer portal accordingly (repository: https://github.com/LedgerHQ/developer-portal).

## coin-modules

Each blockchain family lives in its own package under `libs/coin-modules/coin-<family>`. We should not create a new coin-module if it fits the ecosystem of an existing one. No packages other than coin-modules are allowed in this folder.

Package name: `@ledgerhq/coin-<family>`

### Module directory layout

Every coin module must follow this layout:

| Directory / file | Purpose |
|---|---|
| `api/` | Alpaca API surface — implements `AlpacaApi` from `@ledgerhq/coin-framework/api/types`, types from `@ledgerhq/types-live` are not allowed |
| `logic/` | Core blockchain logic, agnostic of Bridge or API interfaces; only depends on `network/` and external libs, not required if only implementing Alpaca API |
| `network/` | Communication with explorer / indexer / node |
| `types/` | Model definitions for the coin-module, not related to network |
| `bridge/` (legacy) | Bridge implementation (`CurrencyBridge` + `AccountBridge`), relates to types in  `@ledgerhq/types-live` package |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ledger-Wallet-LLC/ledgerwallet](https://github.com/Ledger-Wallet-LLC/ledgerwallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

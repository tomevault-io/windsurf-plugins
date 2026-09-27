---
trigger: always_on
description: Prefer principles, constraints and verifiable outcomes over canned examples.
---

# TiboTattle agent guidance

Prefer principles, constraints and verifiable outcomes over canned examples.

## Use progressive disclosure

- Apply this file to every task in the repository.
- Before changing a scoped area, read the nearest scoped `AGENTS.md`, even when
  the client did not load descendant instructions automatically.
- Read only the references relevant to the task. Do not preload the documentation
  tree, large runbooks, or generated artifacts.
- For unlisted paths, use this root file plus nearby code, manifests, tests, and READMEs.
- If instructions conflict, follow the narrower scope and surface the conflict.

| Work | Read next |
|---|---|
| Product purpose, setup, or repository layout | `README.md` and `CONTRIBUTING.md` |
| Security or privacy-sensitive work | `SECURITY.md` |
| Core product/domain code | `src/AGENTS.md` |
| Codex ingestion or provider normalization | `src/providers/codex/AGENTS.md` |
| Contribution projection or prepared sets | `src/contribution/AGENTS.md` |
| Local companion, dashboard API, or unified index | `apps/local/AGENTS.md` |
| Browser dashboard or public web UI | `apps/web/AGENTS.md` |
| Native macOS, app bundles, updater, or signing | `apps/macos/AGENTS.md` |
| Electron desktop lifecycle, IPC, settings, or shell integration | `apps/electron/AGENTS.md` |
| Hosted Worker, D1/R2, auth, or deployment | `apps/worker/AGENTS.md` |
| Standalone local-review CLI, artifact, install, or deletion | `local-review/AGENTS.md` |
| Exact accounting or price semantics | `packages/accounting/AGENTS.md` |
| Pseudonym derivation or identity continuity | `packages/identity-core/AGENTS.md` |
| Quota calibration, rolling windows, or forecasts | `packages/quota-analysis/AGENTS.md` |
| Locale negotiation, catalogs, or browser i18n mirror | `packages/i18n/AGENTS.md` |
| Telemetry fields, privacy, schemas, or compatibility | `packages/telemetry-contract/AGENTS.md` |
| Native Windows filesystem or credential security | `native/AGENTS.md` |
| GitHub workflows, actions, attestations, or release trust | `.github/AGENTS.md` |
| Build, generator, verification, or release tooling | `scripts/AGENTS.md` |
| Documentation, evidence, plans, or release claims | `docs/AGENTS.md`, then `docs/README.md` |

`CLAUDE.md` imports this policy for all agents.

## Product invariants

Changing an invariant requires an explicit product decision, matching tests,
and updated authoritative documentation.

- Local analysis must work offline, and local HTTP services bind only to loopback.
- Unexpected Keychain security prompts block release. Routine access must be
  non-interactive; never weaken protection to suppress prompts. Follow
  `apps/macos/AGENTS.md` for silent migration, explicit recovery, and proof.
- Prompts, responses, raw commands, credentials, private paths/filenames, raw
  account IDs and session content must not enter derived artifacts, fixtures,
  logs, diagnostics, issues, commits, or PRs. Opt-in Crashpad dumps and explicit
  owner-only raw crash exports are sole local exceptions; never project/upload them (`docs/decisions/2026-09-21-opt-in-crash-doctor.md`).
- Hosted contribution is optional, content-free and pseudonymous. Follow the
  accountless Electron defaults and transition in
  `docs/decisions/2026-09-04-accountless-sharing-policy.md`; preserve durable
  opt-outs and legacy consent contracts; hosted erasure is owner-only.
- Derived data is allowlisted and schema-validated. Unknown upstream fields are
  omitted; unknown, stale, unavailable, or unattributed evidence stays explicit.
  Never convert missing evidence to zero or inferred continuity.
- Accounting and ingestion are replay-safe, identity-scoped, bounded, and
  deterministic. Corrections are additive; durable state transitions are
  recoverable; repeated work must not double count.
- Display windows are not retention policy. Preserve accumulated local evidence
  unless an explicit, receipt-backed deletion workflow authorizes exact targets.
- Schema evolution is fail-closed and forward-moving. Never relabel a database
  version, destructively downgrade state, wipe application data, or let an older
  reader mutate newer state. Diagnose and rehearse recovery against a copy.
- Local, hosted, browser, native, installed-artifact, CI, signed-release, updater,
  and public-deployment evidence are separate gates. Passing one never proves
  another.

## Truth and evidence

- Verify the exact checkout, active diff, relevant runtime, and artifact before
  making a current-state claim.
- Use code and tests to establish implemented behavior, maintained decisions and
  runbooks to establish intended operations, and direct runtime or artifact
  inspection to establish live state. Reconcile disagreements; do not silently
  choose the convenient source.
- Treat dated plans, audits, QA notes, reports, and receipts as point-in-time
  evidence unless `docs/README.md` names them as current authority.
- State the strongest proven result and the remaining gate. Use exact versions,
  commits, paths, commands, and dates when they matter.
- Do not claim provider-authoritative billing, quota formulas, platform support,
  release provenance, or production readiness beyond the evidence actually held.

## Working contract


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamallcock/tibotattle](https://github.com/adamallcock/tibotattle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

---
trigger: always_on
description: UPDATE THIS FILE when making architectural changes, adding patterns, or changing conventions.
---

UPDATE THIS FILE when making architectural changes, adding patterns, or changing conventions.

# Opcore

Opcore is the code-intelligence and robustness monorepo for graph context, edit planning, pre-write validation, repo robustness scanning/measurement, and the standalone ASP Core check provider for coding agents. Remaining old-name env vars, cache dirs, fixtures, receipts, and repo-path names are transitional implementation debt; do not introduce new public/product-facing old-name branding. The accepted runtime/CLI boundary is hybrid: Rust graph core with TypeScript contracts, CLI router, edit, validation, validation-typescript, validation-clone, ASP provider facade, npm/Opcore facade, and ACE descriptors. See @docs/architecture/runtime-cli-ard.md and @docs/planning/opcore-alpha-roadmap.md before changing language, package, provider, product, or CLI ownership.

## Key Concepts

| Concept | Meaning |
|---------|---------|
| Graph provider | Owns source extraction, persistent graph facts, freshness metadata, graph query contracts, and FTS search index artifacts. |
| Edit planner | Owns symbol-aware rename, move, signature, patch, and tree edits; it must validate full edit plans, not isolated files. |
| Validation engine | Owns mechanical checks, hypothetical validation, check manifests, and failure policy. |
| Opcore product facade | Thin user-facing robustness loop over graph, validation, edit, and ASP-provider packages: read-only scan/status/check/measure by default, approval-gated init, and no ASP-standard or old-tool replacement claims. |
| Command adapters | Package-owned graph, edit, check, and validate dispatch surfaces used by canonical `opcore` advanced routes. |
| Current-tool wrappers | Local agent tooling that invokes the already-installed ACE-managed tools; these wrappers are not Opcore implementation artifacts. |

## Where To Look

| Concept | Primary File |
|---------|--------------|
| Runtime/CLI ARD | @docs/architecture/runtime-cli-ard.md |
| Opcore alpha roadmap | @docs/planning/opcore-alpha-roadmap.md |
| Opcore metrics/report/history | `packages/opcore/src/reporting.ts` |
| Latency budgets and trend gate | `docs/performance/latency-budgets.json`, `scripts/check-latency-budgets.mjs` |
| Public contracts | @packages/contracts/ |
| Contract JSON schema | `packages/contracts/schemas/opcore-contracts.schema.json` |
| Command router package | @packages/opcore/src/advanced/ |
| Graph provider package track | @packages/graph/ |
| Graph SQLite store | `crates/graph-core/src/store.rs` |
| Clone graph-core subcommand | `crates/graph-core/src/clone/mod.rs` |
| Edit package track | @packages/edit/ |
| Validation package track | @packages/validation/ |
| Validation file view | `packages/validation/src/overlays.ts` |
| Validation graph client | `packages/validation/src/graph-client.ts` |
| Rust validation adapter | @packages/validation-rust/ |
| TypeScript validation adapter | @packages/validation-typescript/ |
| Clone validation adapter | @packages/validation-clone/ |
| Opcore product facade | @packages/opcore/ |
| Opcore scan report seam | `packages/opcore/src/reporting.ts` |
| ASP Core check provider facade | @packages/asp-provider/ |
| ASP warm inspect/edit session | `packages/opcore/src/advanced/asp-warm/`, @docs/architecture/asp-warm-session-ard.md |
| ASP provider manifest generator | `scripts/write-asp-provider-manifest.mjs` writes canonical `asp-server.json` plus retained provisional install metadata. |
| Golden fixtures and reference evidence | @packages/fixtures/ |
| Graph reference evidence manifest | `packages/fixtures/graph-reference-evidence/manifest.json` |
| Graph release fixture | `packages/fixtures/graph-release/release-readiness-fixture.json` |
| Graph release receipt | `docs/release/graph-release-receipt.json` |
| Graph release payload checksum target | `docs/release/graph-release-receipt.payload.json` |
| Graph release handoff | @docs/release/graph-release-handoff.md |
| Release receipt | `docs/release/release-receipt.json` |
| Release receipt summary | @docs/release/release-receipt.summary.md |
| Cutover receipt | `docs/release/cutover-receipt.json` |
| Cutover receipt summary | @docs/release/cutover-receipt.summary.md |
| ASP dogfood receipt | `docs/release/asp-dogfood-receipt.json` |
| ASP dogfood receipt summary | @docs/release/asp-dogfood-receipt.summary.md |
| Retained guardrail matrix | @docs/release/retained-guardrail-matrix.md |
| Secret scan allowlist | @docs/release/secret-scan-allowlist.json |
| Release receipt generator | `scripts/generate-release-receipt.mjs` |
| Cutover receipt generator | `scripts/generate-cutover-receipt.mjs` |
| ASP dogfood receipt generator | `scripts/generate-asp-dogfood-receipt.mjs` |
| Workspace checks | `scripts/check-workspace.mjs` |
| Package dry-run checks | `scripts/check-packages.mjs` |
| Provenance checks | `scripts/check-provenance.mjs` |
| Current ACE tool setup | `scripts/setup-current-tools.sh` |
| Local CI-equivalent gate | @scripts/ci/run-local-ci-equivalent.sh |
| Zeroshot setup | @.zeroshot/settings.json |
| GitHub Actions | @.github/workflows/ |
| Tests | @tests/ |

## Current Tooling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-open-engine/opcore](https://github.com/the-open-engine/opcore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->

---
trigger: always_on
description: Work in OpenVibeCoding as a contract-first engineering agent:
---

# AGENTS Guide

## Mission

Work in OpenVibeCoding as a contract-first engineering agent:

- keep diffs small and auditable
- keep code and docs in sync
- verify behavior with real commands before claiming success
- never commit runtime caches, local secrets, logs, or generated noise

## Canonical Read Order

1. `README.md`
2. `docs/index.html`
3. `configs/repo_positioning.json`
4. `configs/docs_nav_registry.json`
5. nearest local wrapper under `apps/*/AGENTS.md` or `apps/*/CLAUDE.md`

## Key Commands

- bootstrap: `npm run bootstrap`
- local fast CI: `npm run ci`
- local strict CI: `npm run ci:strict`
- fast verification: `npm run test:quick`
- main local gate: `npm run test`
- host safety scan: `npm run scan:host-process-risks`
- space audit: `npm run space:audit`
- docker runtime audit: `npm run docker:runtime:audit`
- hygiene: `bash scripts/check_repo_hygiene.sh`
- workflow security: `npm run scan:workflow-security`
- Trivy repo scan: `npm run scan:trivy`
- closeout secret scan: `npm run security:scan:closeout`
- truth split: `npm run truth:triage`
- full pre-commit: `pre-commit run --all-files`
- repo-owned `scripts/*.py` entrypoints must remain runnable through direct
  `python3 scripts/<name>.py` execution or `bash scripts/run_governance_py.sh`
  without relying on a pre-seeded repo-root `PYTHONPATH`
- host-compatible pre-commit hooks that execute repo-owned `scripts/*.py`
  entrypoints must use the same wrapper path (or an equivalent `python3 -B`
  contract) so clean hook runs do not leave repo-local `__pycache__` residue

## Generated Governance Context

- active CI layers: `pre-commit`, `pre-push`, `hosted`, `nightly`, `manual`
- do not reintroduce a sixth CI/profile/workflow layer; move scheduled heavy checks to `nightly` and explicit high-cost verification to `manual`

<!-- GENERATED:ci-topology-summary:start -->
- trust flow: `ci-trust-boundary -> quick-feedback -> hosted policy/core slices -> pr-release-critical-gates -> pr-ci-gate`
- hosted policy/core slices: `policy-and-security, core-tests`
- untrusted PR path: `quick-feedback -> untrusted-pr-basic-gates -> pr-ci-gate`
- protected sensitive lanes: `workflow_dispatch -> owner-approved-sensitive -> ui-truth / resilience-and-e2e / release-evidence`
- canonical machine SSOT: `configs/ci_governance_policy.json`
<!-- GENERATED:ci-topology-summary:end -->

<!-- GENERATED:current-run-evidence-summary:start -->
- authoritative release-truth builders must consume `.runtime-cache/openvibecoding/reports/ci/current_run/source_manifest.json`.
- the live current-run authority verdict belongs to `python3 scripts/check_ci_current_run_sources.py` and `.runtime-cache/openvibecoding/reports/ci/current_run/consistency.json`.
- current-run builders: `artifact_index/current_run_index`, `cost_profile`, `runner_health`, `slo`, `portal`, `provenance`.
- docs and wrappers must not hand-maintain live current-run status; they must point readers back to the checker receipts.
- if the current-run source manifest is missing, authoritative current-run reports must fail closed or run only in explicit advisory mode.
<!-- GENERATED:current-run-evidence-summary:end -->

<!-- GENERATED:coverage-summary:start -->
- repo coverage snapshot unavailable
- run `npm run coverage:repo` to refresh this fragment.
<!-- GENERATED:coverage-summary:end -->

## Change Rules

- update tests when behavior changes
- update docs when commands, APIs, or public behavior change
- keep public docs English-first and minimal
- anchor root AI entrypoints to public routes, config manifests, and receipts
  instead of archived internal markdown under `docs/`
- keep runtime output under `.runtime-cache/`
- keep public test/probe fixtures free of maintainer-local absolute paths and
  raw token-shaped literals; use generic workspace roots plus synthetic string
  assembly instead, keep placeholder security URIs constrained to the exact
  `example.com` contract, preserve `.jsonl` temp-report hints in portable scan
  scratch names, fail closed on tracked direct email/phone markers plus
  forbidden runtime files, fail closed on open GitHub secret/code scanning
  alerts in local hooks and pre-push while GitHub-hosted `trusted_pr`,
  `untrusted_pr`, and hosted-first `push_main` Quick Feedback / hosted policy
  lanes stay advisory under integration-token and first-analysis timing
  limits, keep workflow static security (`actionlint` + `zizmor`), canonical
  secret scanning, and Trivy dependency scanning wired into repo-owned
  entrypoints, and sync the root/docs entrypoints when that hygiene contract
  changes
- treat Chrome/Chromium/Playwright/Safari/browser profiles as machine-shared resources:
  use repo-scoped or ephemeral state only, never reuse tabs/windows/profiles
  opened by other repos or other L1s, and close browser sessions when the
  current verification pass ends
- prefer background/no-focus browser verification paths whenever possible; do
  not steal desktop focus unless the current workflow truly requires a visible
  foreground browser step
- before opening a new Chrome/Chromium-class browser instance, check current
  machine browser load; if more than 6 browser instances are already running,
  wait for other repo workers to release resources instead of opening another

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaojiou176-open/OpenVibeCoding](https://github.com/xiaojiou176-open/OpenVibeCoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

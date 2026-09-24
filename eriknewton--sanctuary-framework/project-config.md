---
trigger: always_on
description: This file is a briefing for any AI coding agent working in these codebases. Read it before making any changes. It is the canonical, model-neutral source of truth for how to work in this repo; tool-specific files (for example `CLAUDE.md`) import this file rather than duplicating it.
---

# AGENTS.md: Sanctuary & Concordia Security and Sovereignty Review Context

This file is a briefing for any AI coding agent working in these codebases. Read it before making any changes. It is the canonical, model-neutral source of truth for how to work in this repo; tool-specific files (for example `CLAUDE.md`) import this file rather than duplicating it.

### Attribution Rule (MANDATORY)

**No public-facing document, README, blog post, plugin manifest, package metadata, or software artifact may reference or attribute CIMC as author or creator of Sanctuary or Concordia.** Erik Newton is the sole author. CIMC may be mentioned in internal/biographical context only.

### Commit discipline: test baseline enforcement (MANDATORY)

Every commit to Sanctuary main MUST run `npm run typecheck && npm test` against a clean working tree before staging; block the commit if either fails, if any transform/collection error appears in vitest output, or if the passing-test count drops below the integer in `.test-baseline` at repo root.

**This rule is now backed by structural enforcement, not just instruction:**

- **Pre-commit hook** at `.githooks/pre-commit` runs both gates locally on every `git commit`. Install once with `cd server && npm run install-hooks` (copies the hook into `.git/hooks/pre-commit`). The hook takes ~21 seconds on a modern Mac. Emergency bypass: `SKIP_TEST_BASELINE=1 git commit ...` (logged to `.test-baseline-overrides.log` for audit).
- **CI check** at `.github/workflows/test-baseline-guard.yml` runs the same two gates on every PR and every push to main. This is the second enforcement layer for commits that bypass the local hook with `--no-verify` or from uninstalled environments. See `docs/audit/branch-protection-setup.md` for the Git branch-protection runbook required to make this check a hard merge gate.
- **Written instruction (this block)** remains the human-facing contract. The structural layers make violations hard; this rule makes the intent explicit so a reviewer or auditor can cite it.

See `docs/audit/test-baseline-hardening-plan.md` for the full three-layer hardening plan, `docs/audit/commit-4ac95830-postmortem.md` for the trigger incident, and `docs/audit/branch-protection-setup.md` for the GitHub branch-protection runbook.

### Test isolation: the operator's machine is not a fixture (MANDATORY)

Tests must never read from or write to the operator's real login keychain, real `~/.sanctuary` state, or any other operator-owned credential store. Under test, credential access goes through the keychain chokepoint with an injected in-memory store, so no `security` subprocess is spawned at all. Tests that genuinely exercise keychain integration (including any that spawn the real CLI as a subprocess) use a per-run temporary keychain created in a temp path, scoped to the run's search list, and deleted on teardown; teardown must also reap every server and worker the test spawned, and both must happen even when the test fails or times out. A timed-out test that leaves a live process or a keychain entry behind poisons every later run on the same machine, and the damage looks like unrelated flakes. A structural guard in the suite enforces the no-login-keychain rule; this text records the intent so a reviewer can cite it.

### Assurance discipline: no capability claim without a drill (MANDATORY)

On 2026-08-08 a 46-agent sweep found thirty-two capabilities that were built and unit-tested but never wired into the shipping path, and `ASSURANCE_MATRIX.md` row 17 marking Linux egress enforcement `proven` while the shipped daemon installed no kernel enforcement at all. The row cited integration tests that genuinely pass; they prove the modules, not the shipped boot path. The failure was structural: no rule connected a claim to evidence of the shipped behavior, so a green suite stood in for a shipped path. The first six rules below exist so it cannot recur, and each is structural where a structure is possible, because the convention alone is what failed.

1. **A `proven` platform-enforcement claim requires a drill, not a test suite.** Any `ASSURANCE_MATRIX.md` row whose claim is about enforcement, egress, containment, reboot survival, install, wrapping, jailing, or confinement may read `proven` only if its Evidence cites a captured drill on the platform that matters (a `docs/audit/` artifact or a dated drill record). Test-only or PR-only evidence caps such a row at `partial`. Enforced by `scripts/check-assurance-matrix.mjs` and `.github/workflows/assurance-matrix-guard.yml`. **Absent, indeterminate, or unproven all read as not-proven; none of them reads as passing.**
2. **Every evidence link in the matrix must resolve.** A `proven`/`partial` row pointing at a moved or deleted file is a drift defect. Same guard enforces it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eriknewton/sanctuary-framework](https://github.com/eriknewton/sanctuary-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

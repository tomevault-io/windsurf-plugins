---
trigger: always_on
description: This file defines how automated agents should operate in this repo. Follow it for all changes and PR management.
---

# IntelligenceX Agent Playbook

This file defines how automated agents should operate in this repo. Follow it for all changes and PR management.

**Core Rules**
- Use `gh` CLI for PR, review, and CI workflows.
- Work in a dedicated worktree + branch per task.
- Do not touch the Website PR or website work unless explicitly requested.
- Do not use destructive git commands (`reset --hard`, `checkout --`).

**Worktree + Branch**
1. Create a worktree for every task: `git worktree add -b <branch> <path> origin/master`.
2. Make changes only in that worktree.
3. Keep branches focused to a single change set.

**PR Review and CI**
1. List PRs: `gh pr list --repo EvotecIT/IntelligenceX --state open`.
2. Inspect details: `gh pr view <num> --repo EvotecIT/IntelligenceX`.
3. Check CI: `gh pr checks <num> --repo EvotecIT/IntelligenceX`.
4. Check for unresolved review threads when mergeability stays blocked after checks pass:
   - `gh api graphql -f query='query { repository(owner:"EvotecIT", name:"IntelligenceX") { pullRequest(number:<num>) { reviewThreads(first:100) { nodes { id isResolved isOutdated path line comments(first:10) { nodes { author { login } body url } } } } } } }'`
   - If branch protection requires conversation resolution, unresolved review threads are merge blockers even when the latest review summary says “safe to merge”.
   - Resolve stale threads only after verifying the underlying issue is truly addressed in code; do not auto-resolve live unresolved concerns.
5. If CI fails, inspect logs: `gh run view <run-id> --repo EvotecIT/IntelligenceX --log --job <job-id>`.
6. Classify CI failures before making code changes:
   - Actionable: compilation/test failures, lints, static analysis findings, reviewer bot blockers. Fix in code and re-run checks.
   - Infra-blocked: GitHub billing/spend-limit, runner capacity/unavailable, third-party premium/auth gating (Copilot/Claude/etc), or GitHub outage. Do not churn on code changes. Record the blocker and proceed per the PR Handling Loop infra rule.
7. Do not chase provider/auth diagnostics unless they cause a required check to fail (the reviewer workflow is designed to fail-open for many provider issues).
8. Rebase onto `origin/master` before merge if needed.
9. Merge only when all required checks pass and the PR is mergeable.
10. Merge with squash + delete branch: `gh pr merge <num> --repo EvotecIT/IntelligenceX --squash --delete-branch`.

**Dependabot + Workflow-Only PRs**
- On Dependabot PRs, comments may be authored by `github-actions` rather than the IntelligenceX GitHub App. This is expected: GitHub typically does not expose repo secrets (including app private keys) to Dependabot PR workflows.
- The IntelligenceX reviewer may intentionally skip PRs that only modify workflow files to avoid self-modifying workflow runs. If checks are green and the diff is limited to pin bumps (e.g., `uses:` SHA updates), treat this as mergeable unless maintainers explicitly require a manual review.
- Never touch `/.github/workflows/deploy-website.yml` unless explicitly requested (even when updating pinned action SHAs elsewhere).

**Local Preflight (Before You Push)**
To avoid bot/CI churn, run local checks before pushing when you touched runtime behavior or workflows.
1. Build: `dotnet build IntelligenceX.sln -c Release`
2. Tests: `dotnet test IntelligenceX.sln -c Release`
3. Harness (matches CI): `dotnet ./IntelligenceX.Tests/bin/Release/net8.0/IntelligenceX.Tests.dll`
4. Harness (matches CI): `dotnet ./IntelligenceX.Tests/bin/Release/net10.0/IntelligenceX.Tests.dll`
5. Reviewer static analysis (optional but useful when reviewer logic/templates change):
   - `dotnet run --project IntelligenceX.Cli/IntelligenceX.Cli.csproj --framework net8.0 -- analyze validate-catalog --workspace .`
   - `dotnet run --project IntelligenceX.Cli/IntelligenceX.Cli.csproj --framework net8.0 -- analyze run --config .intelligencex/reviewer.json --out artifacts --framework net8.0`

**PR Handling Loop (Required)**
When an agent is assigned a PR to improve or unblock, it must iterate until merge blockers are clean.

1. Read the latest IntelligenceX bot review comment and snapshot it (copy the raw Markdown to a scratchpad) so you can diff "before vs after" and avoid re-fixing the same item.
2. Treat these sections as merge blockers: `Todo List ✅` and `Critical Issues ⚠️`.
3. Treat `Other Issues 🧯` and `Next Steps 🚀` as non-blocking unless maintainers explicitly escalate them.
4. Triage other automated reviews (for example the “Claude Code Review” sticky comment) and fix anything that impacts correctness, security, or reliability.
5. Fix inline comments only when they map to merge blockers or correctness/security/reliability issues; ignore style-only nits from other bots unless maintainers explicitly escalate them.
6. Apply fixes in a single coherent batch (avoid micro-commits that "poke" the bot repeatedly), then re-run checks and re-check bot output:
   Run: `gh pr checks <num> --repo EvotecIT/IntelligenceX`
   If the bot posts new todo/critical items, repeat.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvotecIT/IntelligenceX](https://github.com/EvotecIT/IntelligenceX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->

---
trigger: always_on
description: Copyright © Advanced Micro Devices, Inc., or its affiliates.
---

<!--
Copyright © Advanced Micro Devices, Inc., or its affiliates.

SPDX-License-Identifier: MIT
-->

---
name: rocm-cli-oss-contribution
description: Safe, high-quality contribution workflow for upstream-facing work in rocm-cli. Activate before any upstream PR, issue, review reply, public fork push, or other external-facing GitHub action.
allowed-tools: Bash(git:*), Bash(gh:*), Bash(cargo:*), Bash(python:*), Bash(make:*), Read, Grep, Glob, Edit, Write
---

# AGENTS: Safe OSS Contribution Workflow For rocm-cli

Use this workflow for any upstream-facing work — PRs, issues, review replies, or public-fork branches — in this or any other public repository.
This is a blocking requirement before upstream-facing activity.

## 0) Activation And Scope

Activate this workflow before any of these actions:

- opening or editing an upstream PR
- opening or editing an upstream issue
- posting or editing an upstream comment or review reply
- pushing a branch intended for upstream

This workflow adds OSS safety and verification guardrails on top of normal dev flow.

## 1) Core Operating Rules

- Solve the reported problem, not an adjacent symptom.
- Be explicit about what is verified versus assumed.
- Never silently skip verification; if a check cannot run, state it clearly.
- Surface options and tradeoffs when multiple approaches are valid.
- Require explicit user approval for high-blast-radius actions:
  - force-pushes
  - opening/closing/merging PRs
  - public comments/review replies
  - changes to shared or public state

## 2) Company Identity Required, Internal Content Forbidden

Identity requirements stay intact:

- use your employer's author/committer identity (when required)
- include DCO sign-off (`git commit -s`)
- keep commit signing enabled when configured; do not bypass with `--no-gpg-sign`, `-n`, or `--no-verify`

Signing and sign-off are now **enforced**, not just policy: local prek hooks
check signing config on every commit (`commit-signing-configured`) and verify
the full range on push (`verify-commits`), and a blocking CI gate
(`commit-signatures`) re-checks every PR with GitHub "Verified" status. See
`docs/commit-signatures.md`.

Content restrictions for upstream surfaces:

- do not include internal/proprietary names, aliases, URLs, hostnames, gateways, cluster names, or registry paths
- do not include links to internal tracking systems or unrelated internal usernames
- bare Jira/EAI-style ticket IDs (e.g. `EAI-1234`) are permitted anywhere this rule applies; do not flag them
- apply this rule to PR titles/bodies, issue text, comments, review replies, commit messages, branch names, code comments, fixtures, and logs

Use neutral external framing (for example: "backend" or "gateway") rather than internal or vendor-specific ownership phrasing.

Leak scan before each upstream push/PR/comment batch:

```bash
INTERNAL_KEYWORDS_PATTERN='internal|confidential|proprietary|private|jira|confluence|\.corp|\.internal'
git diff <upstream-base>..HEAD \
  | grep -inE "$INTERNAL_KEYWORDS_PATTERN" \
  && echo "REVIEW each hit" || echo "diff clean"
```

This default pattern is intentionally generic so the workflow runs as-is. Refine `INTERNAL_KEYWORDS_PATTERN` with your organization's internal names and systems. Also manually review non-diff text surfaces (PR body, comments, issue text, branch name).

## 3) Reproduce First, Then Fix The Actual Issue

- reproduce the reported issue before claiming root cause or fix
- verify that the same reproduction passes after the change
- test boundary paths (non-default config, larger inputs, alternate trigger paths)
- do not relabel an adjacent improvement as "the fix" if the original repro still fails

Every bug fix ships with a regression test in the same change:

- test fails before fix and passes after fix
- choose unit/integration/e2e level based on where the bug lives
- if an e2e cannot run in default CI, state the gap in PR text and cover at another CI level

## 4) Live State Verification Before Any External Claim

Before each stateful decision or public status update:

- refresh remote state (`git fetch`)
- re-check PR status and checks with live `gh` queries
- verify review context against current PR head commit

Do not rely on stale memory, partial CI views, or prior snapshots.
Subagent reports are hypotheses until directly re-verified. When re-verifying, match the verification scope to the claim: if subagent claimed "tests pass", re-run the same test suite; if it claimed "no conflicts", do the rebase locally; if it claimed "leak-free", re-run the scan.

After rebase/cherry-pick/merge, grep for conflict markers:

```bash
grep -rn "^<<<<<<<\|^=======\|^>>>>>>>" .
```

For leak scans, use upstream base `origin/main` (or upstream default branch if different):

```bash
INTERNAL_KEYWORDS_PATTERN='internal|confidential|proprietary|private|jira|confluence|\.corp|\.internal'
git diff origin/main..HEAD \
  | grep -inE "$INTERNAL_KEYWORDS_PATTERN" \
  && echo "REVIEW each hit" || echo "diff clean"
```

Planning and scratch artifacts are not repository documentation. Keep `plans/`,
implementation logs, and agent working notes out of commits; durable design
documentation belongs under `docs/`.

## 5) Investigate rocm-cli Before Editing

Understand existing patterns first:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ROCm/rocm-cli](https://github.com/ROCm/rocm-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->

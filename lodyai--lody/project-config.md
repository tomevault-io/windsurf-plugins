---
trigger: always_on
description: `CLAUDE.md` is a symlink to this file. Edit `AGENTS.md` only.
---

# GitHub contribution automation

`CLAUDE.md` is a symlink to this file. Edit `AGENTS.md` only.

## Ownership

| Area | Source of truth | Contract |
| --- | --- | --- |
| Contributor prompts | `PULL_REQUEST_TEMPLATE.md`, Issue Forms | Ask only for public, actionable contribution context. |
| PR validation | `scripts/check-pr-body.mjs` | Validate the rendered template contract without GitHub mutations. |
| PR reconciliation | `scripts/pr-policy.mjs` | Own disposition, findings, labels, comments, grace period, cleanup, and expiry. |
| Issue linking | `scripts/pr-issue-link.mjs` | Parse and normalize only `## Related issue`. |
| Event orchestration | `workflows/pr-policy.yml`, `workflows/pr-policy-reconcile.yml` | Route every PR event and audit through one concurrency group and one reconciler. |
| Scope labels | `labeler.yml`, `workflows/pr-scope.yml` | Derive configured `scope:*` labels from changed paths. |
| Code checks | `workflows/ci.yml`, `scripts/select-ci-scope.mjs` | Keep `Static checks`/`Tests`. Selector skip/affected fail open; no workflow `paths`. |
| Codex review | root `AGENTS.md` `## Code Review Rules`, `codex-review.md` | Report only P0/P1, security first; 👍 when the linked Issue is solved. |

Do not duplicate a rule across these layers. Changes to required PR template
headings must update the checker in the same commit and validate representative
complete and rejected bodies locally.

Workflow-file security constraints live in
[`workflow-security.md`](workflow-security.md) and bind every change under
`workflows/`.

## Contribution contract

- One-shot identity: Lody team if the user says so, or GitHub login is
  `zxch3n`, `Leeeon233`, or `wibus-wee` (`gh api user --jq .login`, or git
  `user.name` Zixuan Chen, Leon Zhao, or Wibus Wu). Otherwise community; do
  not keep checking. Same-repository branches stay `internal` regardless of
  login.
- Community PRs stay under 1000 changed lines (additions + deletions) unless a
  maintainer assigned the linked Issue to the author. Larger work: file an
  Issue with analysis and wait to be assigned; do not open the PR. Maintainers
  review small focused changes; large unsolicited patches hide invariant breaks.
  Humans: `CONTRIBUTING.md`.
- Draft every PR from `PULL_REQUEST_TEMPLATE.md`; complex changes use `$show-me`.
  Validate external bodies with
  `node .github/scripts/check-pr-body.mjs --body-file <file>`.
- An Agent opens every pull request as a draft (`gh pr create --draft`) and then
  tells its user to mark it ready for review once they judge it ready for
  maintainers. An Agent leaves draft state only when its user asks.
- Every fork-based PR references a Lody Issue and retains the complete Context
  handoff block and its markers. Use `Closes #123` when merging the PR should
  close the Issue and `Refs #123` only when it must stay open. A bare `#123` or
  full Lody Issue URL in `## Related issue` defaults to `Closes #123`.
- Before opening a fork-based PR, an Agent asks its user to publish the authoring
  conversation and fills the required `### Shared conversation` status and link
  or reason using the template. Accept public links from any authoring tool.
  Only the user confirms publication; never invent a URL, refusal, or limitation.
  Wait for an answer: silence is not refusal. If the user declines, preserve their
  exact reply in a separate fenced block under `#### Sharing refusal (verbatim)`
  within `### Original user prompt`, retaining the triggering prompt unchanged.
- Context handoff answers are public evidence, not placeholders. Redact only
  private spans in quoted prompts/replies with explicit markers; entirely
  redacted evidence and `N/A` do not satisfy the contract.
- An Agent preparing a fork-based contribution explains that the Context handoff
  is public and an invalid PR closes after the seven-day correction period.
- Same-repository branches do not create or require an Issue solely for intake,
  and their bodies are not subject to the external contribution template.

## PR policy

`pullRequestDisposition` classifies each current PR before any mutation:

| Disposition | Condition | Behavior |
| --- | --- | --- |
| `bot` | Author login ends in `[bot]` | Do not normalize or enforce; clear prior managed state when present. |
| `bypass` | `status:pr-policy-bypass` is present | Do not normalize or enforce; clear prior managed state when present. |
| `internal` | Numeric `head.repo.id` equals `base.repo.id` | Normalize an explicit Issue reference; do not enforce the external template. |
| `external` | Repository ids differ or either id is missing | Normalize an explicit Issue reference, then enforce the complete contribution contract. |

`author_association` never classifies a PR. A fork remains external when its
author is an owner or member. The bypass label is an explicit maintainer action
whose authorization comes from GitHub's label permissions; the workflow does
not infer identity from PR text. Removing the label immediately resumes normal
classification and enforcement.

Re-read PR details through the API before classification. Body, labels,
repository ids, changed-line totals, and open state must come from the same
current response.

Issue normalization is a stateless, idempotent body edit on human PRs targeting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LodyAI/Lody](https://github.com/LodyAI/Lody) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

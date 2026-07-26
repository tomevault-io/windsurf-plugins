---
trigger: always_on
description: Repository conventions for GitHub Copilot (and any other AI agent reading this file).
---

# Copilot Instructions

Repository conventions for GitHub Copilot (and any other AI agent reading this file).

The **canonical guide is [AGENTS.md](../AGENTS.md)** at the repo root - read it first, including the [PR Review Etiquette](../AGENTS.md#pr-review-etiquette) review-loop contract this file's runbook implements. This file is intentionally narrow: commit/PR-title conventions (summarized inline so VS Code's commit-message and PR-title generators have them) plus the GitHub Copilot Review Runbook.

For code-style rules, see [`CODESTYLE.md`](../CODESTYLE.md) at the repo root.

For PlexCleaner's architecture, processing pipeline, and design patterns, see [../ARCHITECTURE.md](../ARCHITECTURE.md).

Do not duplicate language-specific rules here. **Project-specific conventions and API/behavioral contracts also belong in [AGENTS.md](../AGENTS.md), not here** - this file is intentionally limited to the inline commit/PR-title summary and the GitHub Copilot Review Runbook. Non-Copilot agents (Claude Code, Codex, Cursor, ...) are not directed to this file and don't read it by default, so any rule a reviewer must honor has to live in `AGENTS.md` to be provider-independent.

## Commit Messages and Pull Request Titles

Summarized for VS Code's generators; the full rules, rationale, and examples are in [AGENTS.md "Pull Request Title and Commit Message Conventions"](../AGENTS.md#pull-request-title-and-commit-message-conventions).

- Imperative subject, <= 72 characters, no trailing period; optional blank-line-separated body for the non-obvious *why*.
- US English, title case with lowercase short bind words; no vague titles, no `Co-Authored-By:` unless asked, no release-bump magnitude (NBGV handles versioning). Dependabot's `Bump X from Y to Z` titles are fine.
- develop PRs squash-merge (`gh pr merge --squash`), main PRs merge-commit (`--merge`); a mismatched flag is rejected by branch protection.

## GitHub Copilot Review Runbook

> This runbook implements the [AGENTS.md "PR Review Etiquette"](../AGENTS.md#pr-review-etiquette) review-loop contract for GitHub Copilot. Without it in-repo, an agent has no pointer to the reliable Copilot mechanics and falls back to known-broken paths (the no-op `POST /requested_reviewers`, the wrong bot-login filter). In the API snippets below, `<N>` is the PR number.

Use this section for provider-specific mechanics. The expected review loop *contract* (request review on every push, verify head-SHA coverage, triage findings, reply + resolve, escalate when stuck) is defined in [AGENTS.md -> PR Review Etiquette](../AGENTS.md#pr-review-etiquette). This section only describes how to make GitHub Copilot reliably execute it.

### Triggering and Polling

Auto-review on push is configured (via the branch ruleset's `copilot_code_review` rule with `review_on_push: true`) but fires inconsistently in practice - treat it as best-effort, not guaranteed. After every push, **re-request a review programmatically** via the GraphQL `requestReviews` mutation, passing the Copilot reviewer's bot node id in `botIds`. This drives the loop end-to-end without a UI hand-off.

**A review with no inline comments is still a completed review - not a failure, and not a reason to ask the maintainer to re-trigger.** Copilot very often posts a single formal review (GraphQL `state: COMMENTED`) whose body ends with "...reviewed N of N changed files ... and generated no comments" and adds **zero** inline threads. That review carries the head `commit.oid` and fully satisfies the loop - it is the clean-pass success case. Never read "no inline comments" as "the review didn't run," and never re-request or escalate to the maintainer because comments are absent.

**Round 1 is normally auto-seeded - poll for it before trying to self-trigger.** Auto-review-on-open supplies the first review with no `botIds` call needed, but it can lag one to three minutes. After opening a PR (or the first push), **poll** for a Copilot review on the head SHA (see [Verify Review Covered Current Head](#verify-review-covered-current-head)) before concluding none ran. The `requestReviews` mutation below is for **re-requesting on later pushes** (a new head SHA); by then a prior review exists, so its bot node id is readable. A missing bot node id on round 1 therefore means "the auto-review has not landed yet - wait and poll," **not** "ask the maintainer to kick it off."

> **The reviewer login differs by API.** In **GraphQL** (`gh api graphql` and `gh pr view --json reviews`, which is GraphQL-backed) the `Bot.login` is `copilot-pull-request-reviewer` - **no `[bot]` suffix**. In the **REST** API (`gh api repos/.../issues|pulls/...`) the same account's `user.login` is `copilot-pull-request-reviewer[bot]` - **with** the suffix. Each query below uses the correct form for its API; match the API, not a single spelling, when adapting them.

```sh
# 1. PR node id + the Copilot reviewer's bot node id (read from any existing
#    Copilot review; the reviewer login is `copilot-pull-request-reviewer`).
PR_NODE=$(gh pr view <N> --json id --jq '.id')
BOT_ID=$(gh api graphql -f query='
{
  repository(owner: "ptr727", name: "PlexCleaner") {
    pullRequest(number: <N>) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ptr727/PlexCleaner](https://github.com/ptr727/PlexCleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

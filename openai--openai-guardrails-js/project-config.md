---
trigger: always_on
description: When submitting a PR, always monitor CI for failures. Automatically fix and re-push only failures introduced or worsened by the change, or narrowly necessary to achieve the requested outcome. Report unrelated preexisting failures and retry transient or flaky checks when appropriate; do not expand the diff to fix them. Once CI passes, post in #sdk-reviews and ask for a review. Always post in the root #sdk-reviews channel, do not post in threads.
---

# Repository instructions

When submitting a PR, always monitor CI for failures. Automatically fix and re-push only failures introduced or worsened by the change, or narrowly necessary to achieve the requested outcome. Report unrelated preexisting failures and retry transient or flaky checks when appropriate; do not expand the diff to fix them. Once CI passes, post in #sdk-reviews and ask for a review. Always post in the root #sdk-reviews channel, do not post in threads.

When addressing feedback on a PR - always leave a comment describing how you fixed the particular issue, and then resolve the comment after pushing.

Before pushing code, opening a pull request, or updating an existing pull request, always complete the adversarial-review procedure below. Use $adversarial-review when that skill is available; otherwise follow this inline procedure directly. Before implementation, define the original requested outcome, acceptance criteria, affected code paths, and explicit non-goals. Obtain user approval before materially expanding the diff, crossing unrelated ownership boundaries, changing public APIs, or restructuring architecture. For each adversarial-review round, explicitly spawn exactly two independent, read-only subagents with fork_turns="none" so neither inherits the parent conversation or the other reviewer's analysis. Give each reviewer a self-contained brief with the exact worktree path, current HEAD SHA, comparison base SHA, original user-requested outcome, acceptance criteria, and explicit non-goals. Both must independently review the complete changes, including branch commits, staged and unstaged changes, and relevant untracked files, in that same worktree. Reviewers must not edit files, modify Git state, or spawn additional agents. If fresh-context subagents are unavailable, stop and report the limitation before pushing or updating the PR. Do not create separate Codex tasks or additional Git worktrees. Aggregate their findings and fix only supported issues introduced or worsened by the change, or narrowly necessary to achieve the requested outcome correctly and safely. Report unrelated preexisting defects, broader cleanup, and architectural improvements as separate follow-up recommendations; they must not expand the PR or prevent review convergence. Repeat with two newly spawned fresh-context reviewers per round until two consecutive rounds produce no meaningful, unresolved, in-scope blocking findings. Perform relevant testing and run applicable linters. If the change touches any security surfaces, perform a security review. Do not push or open/update a pull request before these checks are complete. If review has not converged after ten rounds, stop and report the remaining issues. Deeply scrutinize the requested change without expanding its scope.

## Task scope and review discipline

Before implementing or reviewing a change, identify the specific user-requested outcome and acceptance criteria, the code paths and tests reasonably necessary to achieve them, and explicit non-goals. Every changed file and behavior must be justified by that outcome, a regression introduced or worsened by the change, or a narrowly necessary prerequisite.

Do not fix unrelated preexisting bugs, modernize surrounding code, expand tests for unrelated behavior, redesign APIs, introduce general-purpose abstractions, or restructure neighboring modules merely because review uncovers an opportunity. Classify each finding as an introduced or worsened defect, a narrowly necessary correction, a preexisting unrelated problem, a broader improvement, or a serious concern that requires user agreement before proceeding. Fix only the first two categories in the current PR; report the next two separately without creating external issues or additional work unless requested, and stop for user agreement on the last.

Prefer the smallest coherent fix. If addressing feedback would substantially increase the diff, touch unrelated ownership boundaries, change public APIs, or require architectural restructuring, stop and request approval before expanding scope. Scope expansion is itself a code-quality regression. A clean review round has no unresolved, supported, in-scope blocking findings; out-of-scope observations never prevent convergence.

When writing or modifying tests - prefer code that satisfies the linter over adding inline lint suppressions. Never add a suppression when a straightforward compliant form exists; if a suppression is genuinely necessary, document why.

Prefer Vitest mocks or spies over ad-hoc test doubles when they exercise the real interface correctly. Verify that the mock matches the interface consumed by the code under test; otherwise use the smallest concrete implementation and explain the constraint in review feedback.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openai/openai-guardrails-js](https://github.com/openai/openai-guardrails-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

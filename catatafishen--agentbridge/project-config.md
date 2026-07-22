---
trigger: always_on
description: This plugin follows two core principles. See [docs/DESIGN-PRINCIPLES.md](docs/DESIGN-PRINCIPLES.md)
---


# Design Principles

This plugin follows two core principles. See [docs/DESIGN-PRINCIPLES.md](docs/DESIGN-PRINCIPLES.md)
for detailed rationale and examples.

**1. Internally: prefer JetBrains APIs over custom code.**
IntelliJ already provides OS detection (`SystemInfo`), shell management (`TerminalProjectOptionsProvider`), VCS
integration (`git4idea`), project model, SDK resolution, UI threading (`ApplicationManager.invokeLater`), and much more.
Never reimplement what the platform already does — our version will be less robust and may look suspicious to users.

**2. MCP tools: be a bridge, not an inventor.**
Every MCP tool should wrap an IntelliJ action. If JetBrains provides a feature, proxy it. If JetBrains doesn't provide
it, decide whether it belongs here at all. Never build custom implementations (raw JDBC, subprocess scanning, etc.) as
substitutes for IDE-level features. If a feature isn't available in the user's IDE installation, disable the tool
gracefully — the agent can use a specialist MCP server instead.

# Development Workflow

Each feature or bug fix must be done in its own branch and a PR created when the work is done.

- Branch name: `feat/<short-description>` or `fix/<short-description>` (matching the commit convention)
- One logical change per branch — do not bundle unrelated changes
- Create a PR as soon as the branch is ready for review
- Do not commit directly to `master`

## Do Not Fan Out Branches While Work Is Unmerged

**When an earlier PR from the current session is still unmerged, do not start a new branch for a
follow-up change. Add follow-up commits to the same branch (or reopen the branch's PR scope) unless
the human explicitly asks for a separate PR.**

Parallel branches on related code cause real damage in an agent-driven workflow:

- **Merge conflicts** — the same file gets edited on two branches, and one merge overwrites or fights
  the other's changes.
- **Duplicate work** — a reviewer's feedback on PR A is fixed independently on branch B, and both
  branches end up "fixing" the same thing in slightly different ways.
- **Rebase debt** — the second branch is based on an older master and has to be rebased once the
  first merges, often with conflicts.
- **Review fragmentation** — a coherent behaviour (e.g. "the Stop button works properly") gets split
  across 2–3 PRs that reviewers have to context-switch between.

**The rule:**

1. **Default to extending the current branch.** If the new fix is in the same area of the codebase
   or logically follows from work you just did in an open PR, add a commit to that branch.
2. **Ask before branching.** If you genuinely believe a separate branch is warranted (e.g. the fix
   is fully independent, or the current PR is close to merge and you don't want to expand its
   scope), state your reasoning and ask the human to confirm before creating the new branch.
3. **Never open a second branch on the same file** that another open PR of yours already modifies —
   unless the human has explicitly told you the first PR is finalised.
4. **When adding to an existing PR's branch,** update the PR title and description to reflect the
   expanded scope so reviewers know what they are getting.

The bias should be toward *fewer, larger, coherent PRs* rather than *more, smaller, parallel PRs* —
because agent throughput is fast enough that parallel branches conflict faster than reviewers can
merge them.

## Build and Test Verification

**After making code changes**, always run `build_project` and verify it succeeds before ending
the turn. Do not consider the work done if the build is red — fix all errors first.

**Before pushing to a branch that has an open PR** (or before creating a PR), run `run_tests`
and verify all unit tests pass. A failing test suite blocks review; fix failures before pushing.

## Async CI Pattern

CI takes 3-5 minutes after each push or rebase. Do not wait for it synchronously — check
`gh pr checks <number>` between other tasks. A good cadence: push a branch, work on something else, come back to verify
once CI has had time to finish.

If CI is failing: `gh pr checks <number>` to see which check failed, then
`gh run view <run-id> --log-failed` for details.

## Handling PR Review Comments

When a PR has review comments, for **each comment thread**:

1. **Address the concern** — fix the code, or decide not to fix it (with justification).
2. **Reply to the thread** explaining what was done (or why it was intentionally left as-is):
   ```
   gh api repos/catatafishen/agentbridge/pulls/COMMENTS/COMMENT_ID/replies \
     -f body="Fixed: description of what was changed and where."
   ```
3. **Mark the thread as resolved** via the GitHub GraphQL API:
   ```
   # Step 1 — get thread node IDs
   gh api graphql -f query='{
     repository(owner:"catatafishen", name:"agentbridge") {
       pullRequest(number:N) {
         reviewThreads(first:50) { nodes { id isResolved } }
       }
     }
   }' --jq '.data.repository.pullRequest.reviewThreads.nodes[]'

   # Step 2 — resolve each unresolved thread
   gh api graphql -f query='mutation {
     resolveReviewThread(input: {threadId: "PRRT_..."}) {
       thread { id isResolved }
     }
   }'
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catatafishen/agentbridge](https://github.com/catatafishen/agentbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

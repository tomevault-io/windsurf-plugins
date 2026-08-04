---
trigger: always_on
description: Guidelines for AI agents working on this codebase.
---

# Apache Camel Kamelets - AI Agent Guidelines

Guidelines for AI agents working on this codebase.

## Project Info

The Apache Camel Kamelet Catalog is the default catalog of reusable Camel route
templates ("Kamelets"), distributed as Kubernetes-style YAML and consumed by
Camel, Camel K, Camel Quarkus and Camel Spring Boot.

- Version: 4.20.0
- Camel: 4.20.0
- Java: 17+
- Build: Maven (`mvn verify`); Go is required for the `script/` and `crds/`
  generators
- Issue tracker: GitHub — https://github.com/apache/camel-kamelets/issues
- Related repository: `apache/camel` (Camel core — the Kamelet **runtime**
  lives there, not here)

## AI Agent Rules of Engagement

These rules apply to ALL AI agents working on this codebase.

### Attribution

- All AI-generated content (GitHub PR descriptions, review comments, issue
  comments) MUST clearly identify itself as AI-generated and mention the human
  operator. Example: "_Claude Code on behalf of [Human Name]_"

### PR Volume

- An agent MUST NOT open more than 10 PRs per day per operator to ensure human
  reviewers can keep up.
- Prioritize quality over quantity — fewer well-tested PRs are better than many
  shallow ones.

### Git branch

- An agent MUST NEVER push commits to a branch it did not create.
- If a contributor's PR needs changes, the agent may suggest changes via review
  comments, but must not push to their branch without explicit permission.
- An agent should prefer to use its own fork to push branches instead of the
  main `apache/camel-kamelets` repository, to avoid filling the main repository
  with uncleaned branches.
- Branch names: fix-issue → `ci-issue-<ISSUE_NUMBER>`, quick-fix →
  `quick-fix/<short-slug>`, CI fix → `ci-fix/<short-slug>`. Include the topic
  and issue number where possible.
- After a Pull Request is merged or rejected, delete the branch.

### GitHub Issue Ownership

- An agent MUST ONLY pick up **unassigned** GitHub issues.
- If an issue is already assigned to a human, the agent must not reassign it or
  work on it.
- Before starting work, assign the issue to its operator.
- Beginner tasks carry the `good first issue` label; experienced tasks carry
  `help wanted`.

### Commits

- Fix an issue: `Fix #<ISSUE_NUMBER>: <brief description>`
- Quick fix / chore: `chore: <brief description>`
- CI fix: `ci: <brief description>`
- Always reference the GitHub issue when applicable. The repository accepts
  **squash** and **rebase** merges only (no merge commits).

### PR Description Maintenance

When pushing new commits to a PR, **always update the PR description** (and
title if needed) to reflect the current state of the changeset. Use
`gh pr edit --title "..." --body "..."` after each push.

### PR Reviewers

When creating a PR, **always identify and request reviews** from the most
relevant committers:

- Run `git log --format='%an' --since='1 year' -- <affected-files> | sort | uniq -c | sort -rn | head -10`
  to find who has been most active on the affected files.
- Use `git blame` on key modified files to identify who wrote the code.
- Cross-reference with the
  [committer list](https://home.apache.org/committers-by-project.html#camel).
- Request review from **at least 2 relevant committers** via
  `gh pr edit --add-reviewer`.
- When all comments are addressed and checks are green, re-request review.

### Merge Requirements

- An agent MUST NOT merge a PR if there are any **unresolved review
  conversations**.
- An agent MUST NOT merge a PR without at least **one human approval**.
- An agent MUST NOT approve its own PRs — human review is always required.

### Code Quality

- Every new or changed Kamelet SHOULD include Citrus tests under
  `tests/camel-kamelets-itest/src/test/resources/<kamelet-name>/`; Kamelets with
  passing behaviour tests are labelled `camel.apache.org/kamelet.verified=true`.
- After adding or modifying a Kamelet, **regenerate and commit** the generated
  docs and validate, or CI will fail:
  ```bash
  cd script/generator && go run . ../../kamelets/ ../../docs/modules/ROOT/
  cd ../validator     && go run . ../../kamelets/      # must report no ERRORS
  ```
- `nav.adoc` and the per-Kamelet doc pages are **generated** — do not hand-edit
  them.
- A full `mvn verify` from the repository root must pass before pushing.

### Asynchronous Testing

Do **NOT** use `Thread.sleep()` in test code; it leads to flaky, slow,
non-deterministic tests. Use the project's Citrus test constructs (or
Awaitility, where Java test code applies) with an explicit timeout instead.

### Issue Investigation (Before Implementation)

Before implementing a fix, **thoroughly investigate** the issue. Kamelets are a
long-lived shared catalog — a template often looks "wrong" but exists for a
reason (compatibility with a Camel component default, an explicit insecure
convenience Kamelet, an intentional inbound-header mapping).

1. **Validate the issue** — confirm it is real and reproducible; question
   assumptions in the description.
2. **Check git history** — `git log --oneline <file>` and `git blame <file>`;
   read commit messages and linked issues.
3. **Search for related issues/PRs** on GitHub for prior discussion or
   intentional decisions.
4. **Check the developer guide** — `docs/modules/ROOT/pages/development.adoc`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/camel-kamelets](https://github.com/apache/camel-kamelets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

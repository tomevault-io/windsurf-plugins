---
trigger: always_on
description: Guidelines for AI agents working on this codebase.
---

# Apache Camel - AI Agent Guidelines

Guidelines for AI agents working on this codebase.

## Project Info

Apache Camel is an integration framework supporting routing rules in Java, XML and YAML DSLs.

- Version: 4.22.0-SNAPSHOT
- Java: 17+
- Build: Maven 3.9.12+

## AI Agent Rules of Engagement

These rules apply to ALL AI agents working on this codebase.

### Attribution

- All AI-generated content (GitHub PR descriptions, review comments, JIRA comments) MUST clearly
  identify itself as AI-generated and mention the human operator.
  Example: "_Claude Code on behalf of [Human Name]_"
- **Never guess or hallucinate the operator's name.** Always determine it programmatically:
  - Use `gh api /user --jq '.login'` to get the authenticated GitHub username.
  - If for any reason the lookup fails, omit the name rather than guessing.
- AI coding agents MUST be configured to add co-authorship trailers to commits
  (e.g., `Co-authored-by`). For Claude Code, enable this via the
  [attribution settings](https://code.claude.com/docs/en/settings#attribution-settings).

### PR Volume

- An agent MUST NOT open more than 10 PRs per day per operator to ensure human reviewers can keep up.
- Prioritize quality over quantity — fewer well-tested PRs are better than many shallow ones.

### Git branch

- An agent MUST NEVER push commits to a branch it did not create.
- If a contributor's PR needs changes, the agent may suggest changes via review comments,
  but must not push to their branch without explicit permission.
- An agent should prefer to use his own fork to push branches instead of the main apache/camel repository. It avoids to fill the main repository with a long list of uncleaned branches.
- An agent must provide a useful name for the git branch. It should contain the global topic and issue number if possible.
- After a Pull Request is merged or rejected, the branch should be deleted.

### JIRA Ticket Ownership

- An agent MUST ONLY pick up **Unassigned** JIRA tickets.
- If a ticket is already assigned to a human, the agent must not reassign it or work on it.
- Before starting work, the agent must assign the ticket to its operator and transition it to "In Progress".
- Before closing a ticket, always set the correct `fixVersions` field.
  Note: `fixVersions` cannot be set on an already-closed issue — set it before closing,
  or reopen/set/close if needed.

### PR Description Maintenance

When pushing new commits to a PR, **always update the PR description** (and title if needed) to
reflect the current state of the changeset. PRs evolve across commits — the description must stay
accurate and complete. Use `gh pr edit --title "..." --body "..."` after each push.

### PR Reviewers

When a PR is **ready for review** (not in draft), **always identify and request reviews** from
the most relevant committers. **Do NOT request reviewers on draft PRs** — wait until the PR is
marked ready for review.

- Run `git log --format='%an' --since='1 year' -- <affected-files> | sort | uniq -c | sort -rn | head -10`
  to find who has been most active on the affected files.
- Use `git blame` on key modified files to identify who wrote the code being changed.
- Cross-reference with the [committer list](https://home.apache.org/committers-by-project.html#camel)
  to ensure you request reviews from active committers (not just contributors).
- For component-specific changes, prefer reviewers who have recently worked on that component.
- For cross-cutting changes (core, API), include committers with broader project knowledge.
- Request review from **at least 2 relevant committers** using `gh pr edit --add-reviewer`.
- When all comments on the Pull Request are addressed (by providing a fix or providing more explanation) and the PR checks are green, re-request review on existing reviewers so that they are aware that the new changeset is ready to be reviewed.

### Doing a review

When an AI agent is doing a review:

- Wait until PR checks are green as they will already catch most trivial issues using less resources
- It must challenge the code and ensure that it respects all conventions
- For Dependabot PRs, either do not review them or be able to do a real review: check for deprecated APIs, removed features, or breaking changes in the changelog

### Merge Requirements

- An agent MUST NOT merge a PR if there are any **unresolved review conversations**.
- An agent MUST NOT merge a PR without at least **one human approval**.
- An agent MUST NOT approve its own PRs — human review is always required.

### Merge Procedure

When merging a PR, an agent MUST perform the following steps **in order**:

1. **Derive the milestone from the target branch**:
   - Read the `<version>` from the root `pom.xml` on the PR's **target branch** (e.g., `main`,
     `camel-4.18.x`).
   - Strip the `-SNAPSHOT` suffix to get the milestone name (e.g., `4.22.0-SNAPSHOT` → `4.22.0`).

2. **Assign the milestone**:
   - Set the GitHub milestone on the PR: `gh pr edit <PR> --milestone <version>`.
   - If the milestone does not exist yet on GitHub, create it first:
     `gh api repos/{owner}/{repo}/milestones -f title="<version>"`.
   - Set `fixVersions` on the corresponding JIRA issue to the same version. Note: `fixVersions`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/camel](https://github.com/apache/camel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

---
trigger: always_on
description: Guidelines for AI agents working on this codebase.
---

# Apache Camel Spring Boot - AI Agent Guidelines

Guidelines for AI agents working on this codebase.

## Project Info

Apache Camel Spring Boot provides Spring Boot auto-configuration and starter modules for Apache Camel components.

- Version: 4.22.0-SNAPSHOT
- Java: 17+
- Build: Maven (use `./mvnw` wrapper)
- Issue tracker: [JIRA (CAMEL project)](https://issues.apache.org/jira/browse/CAMEL)
- Related repository: [apache/camel](https://github.com/apache/camel) — Camel core

## AI Agent Rules of Engagement

These rules apply to ALL AI agents working on this codebase.

### Attribution

- All AI-generated content (GitHub PR descriptions, review comments, JIRA comments) MUST clearly
  identify itself as AI-generated and mention the human operator.
  Example: "_Claude Code on behalf of [Human Name]_"

### PR Volume

- An agent MUST NOT open more than 10 PRs per day per operator to ensure human reviewers can keep up.
- Prioritize quality over quantity — fewer well-tested PRs are better than many shallow ones.

### Git branch

- An agent MUST NEVER push commits to a branch it did not create.
- If a contributor's PR needs changes, the agent may suggest changes via review comments,
  but must not push to their branch without explicit permission.
- An agent should prefer to use his own fork to push branches instead of the main
  apache/camel-spring-boot repository. It avoids filling the main repository with a long list
  of uncleaned branches.
- An agent must provide a useful name for the git branch. It should contain the global topic
  and issue number if possible.
- After a Pull Request is merged or rejected, the branch should be deleted.

### JIRA Ticket Ownership

- An agent MUST ONLY pick up **Unassigned** JIRA tickets.
- If a ticket is already assigned to a human, the agent must not reassign it or work on it.
- Before starting work, the agent must assign the ticket to its operator and transition it to
  "In Progress".
- Before closing a ticket, always set the correct `fixVersions` field.
  Note: `fixVersions` cannot be set on an already-closed issue — set it before closing,
  or reopen/set/close if needed.

### PR Description Maintenance

When pushing new commits to a PR, **always update the PR description** (and title if needed) to
reflect the current state of the changeset. PRs evolve across commits — the description must stay
accurate and complete. Use `gh pr edit --title "..." --body "..."` after each push.

### PR Reviewers

When creating a PR, **always identify and request reviews** from the most relevant committers:

- Run `git log --format='%an' --since='1 year' -- <affected-files> | sort | uniq -c | sort -rn | head -10`
  to find who has been most active on the affected files.
- Use `git blame` on key modified files to identify who wrote the code being changed.
- Cross-reference with the [committer list](https://home.apache.org/committers-by-project.html#camel)
  to ensure you request reviews from active committers (not just contributors).
- For component-specific changes, prefer reviewers who have recently worked on that component.
- For cross-cutting changes (core, auto-configuration), include committers with broader project
  knowledge.
- Request review from **at least 2 relevant committers** using `gh pr edit --add-reviewer`.
- When all comments on the Pull Request are addressed (by providing a fix or providing more
  explanation) and the PR checks are green, re-request review on existing reviewers so that they
  are aware that the new changeset is ready to be reviewed.

### Merge Requirements

- An agent MUST NOT merge a PR if there are any **unresolved review conversations**.
- An agent MUST NOT merge a PR without at least **one human approval**.
- An agent MUST NOT approve its own PRs — human review is always required.

### Code Quality

- Every PR must include tests for new functionality or bug fixes.
- Every PR must include documentation updates where applicable.
- All generated files must be regenerated and committed (CI checks for uncommitted changes).

### Asynchronous Testing: Use Awaitility Instead of Thread.sleep

Do **NOT** use `Thread.sleep()` in test code. It leads to flaky, slow, and non-deterministic tests.
Use the [Awaitility](https://github.com/awaitility/awaitility) library instead, which is already
available as a test dependency in the project.

**Example — waiting for a route to be registered:**

```java
import static org.awaitility.Awaitility.await;

await().atMost(20, TimeUnit.SECONDS)
       .untilAsserted(() -> assertEquals(1, context.getRoutes().size()));
```

**Rules:**

- New test code MUST NOT introduce `Thread.sleep()` calls.
- When modifying existing test code that contains `Thread.sleep()`, migrate it to Awaitility.
- Always set an explicit `atMost` timeout to avoid hanging builds.
- Use `untilAsserted` or `until` with a clear predicate — do not replace a sleep with a
  busy-wait loop.

### Issue Investigation (Before Implementation)

Before implementing a fix for a JIRA issue, **thoroughly investigate** the issue's validity and
context. Camel is a large, long-lived project — code often looks "wrong" but exists for good
reasons. Do NOT jump straight to implementation after reading the issue description and the
current code.

**Required investigation steps:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/camel-spring-boot](https://github.com/apache/camel-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

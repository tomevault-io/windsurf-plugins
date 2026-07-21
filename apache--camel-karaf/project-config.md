---
trigger: always_on
description: Licensed to the Apache Software Foundation (ASF) under one
---

<!--
    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.
-->

# Apache Camel Karaf - AI Agent Guidelines

Guidelines for AI agents working on this codebase.

## Project Info

Apache Camel Karaf provides Apache Karaf (OSGi) runtime support for Apache
Camel: Karaf feature descriptors that install Camel and its components as OSGi
bundles, an OSGi-aware core (`camel-core-osgi`), a Blueprint XML DSL
(`camel-blueprint`), and Karaf shell commands for inspecting and controlling
running Camel contexts and routes.

- Version: 4.18.2-SNAPSHOT
- Bundles Apache Camel: 4.18.1
- Built against Apache Karaf: 4.4.8 (minimum Karaf 4.4.6)
- Java: 17+
- Build: Maven

It is a sub-project of Apache Camel, under the same PMC and the same security
process.

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
  main `apache/camel-karaf` repository. It avoids filling the main repository
  with a long list of uncleaned branches.
- An agent must provide a useful name for the git branch. It should contain the
  global topic and issue number if possible.
- After a Pull Request is merged or rejected, the branch should be deleted.
- `main` requires **linear history** and PRs are **squash-merged** (see
  `.asf.yaml`). Keep branches rebased on `main`; do not add merge commits.

### GitHub Issue Ownership

camel-karaf tracks issues on **GitHub** (not JIRA):
<https://github.com/apache/camel-karaf/issues>.

- An agent MUST ONLY pick up issues that are **unassigned**.
- If an issue is already assigned to a human, the agent must not reassign it or
  work on it.
- Before starting work, the agent should comment on the issue stating it (and
  its operator) is taking it, so work is not duplicated.
- Reference the issue from the PR so it is linked and auto-closed on merge.

### PR Description Maintenance

When pushing new commits to a PR, **always update the PR description** (and
title if needed) to reflect the current state of the changeset. Use
`gh pr edit --title "..." --body "..."` after each push.

### PR Reviewers

When creating a PR, **always identify and request reviews** from the most
relevant committers:

- Run `git log --format='%an' --since='1 year' -- <affected-files> | sort | uniq -c | sort -rn | head -10`
  to find who has been most active on the affected files.
- Use `git blame` on key modified files to identify who wrote the code being
  changed.
- Cross-reference with the
  [committer list](https://home.apache.org/committers-by-project.html#camel)
  to ensure you request reviews from active committers.
- Request review from **at least 2 relevant committers** using
  `gh pr edit --add-reviewer`.
- When all comments are addressed and checks are green, re-request review so
  reviewers know the new changeset is ready.

### Merge Requirements

- An agent MUST NOT merge a PR if there are any **unresolved review
  conversations**.
- An agent MUST NOT merge a PR without at least **one human approval**.
- An agent MUST NOT approve its own PRs — human review is always required.

### Code Quality

- Every PR must include tests for new functionality or bug fixes.
- Every PR must include documentation updates where applicable.
- The build (`mvn clean install`) is the gate; it must pass before pushing.
  There is no separate enforced source-formatter plugin — match the style of
  the surrounding code and the Apache Camel conventions.
- Any generated files (catalog, features) must be regenerated and committed if
  the change affects them.

### Asynchronous Testing: Use Awaitility Instead of Thread.sleep

Do **NOT** use `Thread.sleep()` in test code. It leads to flaky, slow, and
non-deterministic tests. Use the
[Awaitility](https://github.com/awaitility/awaitility) library instead, which

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/camel-karaf](https://github.com/apache/camel-karaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

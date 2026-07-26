---
trigger: always_on
description: This file is auto-synced to all repositories across the enterprise.
---

# GridLtd Organization Rules

This file is auto-synced to all repositories across the enterprise.
Do not edit directly — update the source at Gridltd-DevOps/.github/org-rules.md

## Team Directory

When you need to look up team members, fetch the current internal team
directory before acting:

```bash
gh api repos/Gridltd-DevOps/.github/contents/team/members.yml -q '.content' | base64 -d
```

Identity rules:

- `github` is the stable GitHub login. Use it for GitHub operations: assign issues/PRs, request reviews, mention users, CODEOWNERS, branch/commit attribution checks.
- `lark_open_id` is the stable Feishu/Lark user id. Use it for Feishu direct messages or user mentions.
- `email` is the stable email destination. Use it for mail only.
- `github_name` is GitHub display metadata and is not stable or guaranteed unique.
- `alias`, `name`, `role`, `responsibility`, and `profile` are human-readable metadata only; never use them directly as external action identifiers.

If the user gives only a name, alias, display name, role, or other ambiguous
human label, resolve it against `team/members.yml` first. If more than one
member matches, or if the match depends on memory rather than the current file,
pause and ask the user to confirm the exact `github`, `lark_open_id`, or
`email` before taking any external side effect.

Preferred deterministic resolver:

```bash
python .github/workflows/scripts/members/resolve_team_member.py --members-file team/members.yml --query "<name-or-alias-or-login>"
```

Use `--check` before changing the directory or agent identity rules:

```bash
python .github/workflows/scripts/members/resolve_team_member.py --members-file team/members.yml --check
```

## Organization Standards

### Communication

- Communicate in Chinese, keep technical terms in English
- Commit messages and PR descriptions must be in English

### Git Flow Branching Model

| Branch | Purpose | Merges to |
|--------|---------|-----------|
| `main` | Production-ready code, always stable | — |
| `develop` | Integration branch for features | `main` (via release) |
| `feature/<issue-id>-<short-desc>` | New features | `develop` |
| `release/<version>` | Release preparation & QA | `main` + `develop` |
| `hotfix/<issue-id>-<short-desc>` | Urgent production fixes | `main` + `develop` |

- Branch names use lowercase + hyphens, e.g. `feature/42-user-auth`
- Delete branches after merge

### Commit Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

| Type | Usage |
|------|-------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code restructuring (no behavior change) |
| `docs` | Documentation only |
| `chore` | Build, CI, dependencies, tooling |
| `test` | Add or update tests |
| `style` | Formatting, whitespace (no logic change) |
| `perf` | Performance improvement |

- `scope` should reflect the module or area, e.g. `feat(auth): add SSO login`
- Breaking changes: add `BREAKING CHANGE:` in footer or `!` after type, e.g. `feat(api)!: remove v1 endpoints`

### Pull Requests

- PR title follows the same Conventional Commits format
- Current hard PR gate requires exact `## Related Issue`, non-empty
  `## Summary`, non-empty `## Test Plan`, and a valid issue link keyword.
- Use `Closes`, `Fixes`, or `Resolves` when the PR completes the issue. Use
  `Refs` only for epic, cross-cutting, ADR, roadmap, ledger, or advisory parent
  work that intentionally remains open.
- PR governance reads GitHub-native metadata first: title, base/head branch,
  draft state, labels, assignees, review requests, review decision, merge
  state, status checks, closing issues, parent/sub-issues, dependencies, and
  Project fields when scope is available. PR body is review evidence and
  fallback, not the sole source of truth.
- New PR metadata sections are advisory until explicitly promoted through the
  approval path: `## Native Metadata`, `## Change Scope`, `## Impact`,
  `## AC Verification`, `## Rollback`, `## Risk`, and
  `## Agent Exposure Block`.
- `## Agent Exposure Block` is required for Agent-authored, Agent-executed, or
  materially Agent-edited PRs once the repository validator requires it. Until
  then, Agents should add it whenever they touch a PR.
- Require at least 1 approval before merge
- Use **Squash Merge** for feature → develop; **Merge Commit** for release/hotfix → main
- Keep PRs focused — one concern per PR

### Issue Creation

When creating issues (via CLI, API, or any Agent tool), you MUST follow the
organization issue templates. The issue gate validates API/CLI-created bodies as
well as form-created bodies, adds `needs-template` or `needs-info` on failure,
and comments with deterministic remediation. Stale `needs-template` cleanup is a
separate conservative workflow and is not a substitute for creating compliant
issues.

Execution-ready issues must keep Discovery and Delivery separated. Raw
Research, Discussion, or RFC issues may contain rejected assumptions and must not
be used as the direct closing source for Delivery PRs. Delivery work must link
an accepted Requirement / PRD, Initiative, Bug, Regression, Incident, Test
failure, operational exception, or approval issue.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NarratorAI-Studio/narrator-ai-cli](https://github.com/NarratorAI-Studio/narrator-ai-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

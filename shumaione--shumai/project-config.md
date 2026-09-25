---
trigger: always_on
description: We use a pull request–based workflow for all tasks.
---

# Developer Guide

We use a pull request–based workflow for all tasks.

## Workflow

Before starting any task, create and switch to a new feature branch:

```bash
git checkout -b <branch-name>
```

Complete the task on that branch.

After finishing the work and verifying that all checks pass:

1. Update `CHANGELOG.md` under `## [Unreleased]` for user-facing changes (see [Changelog Guidelines](#changelog-guidelines)).
2. Stage and commit your changes.
3. Push the branch to `origin`.
4. Open a pull request using the GitHub CLI.

```bash
git add .
git commit -m "<commit-message>"
git push origin <branch-name>
gh pr create --title "<pull-request-title>" --body "<pull-request-body>"
```

---

## Commit Messages

We follow the Conventional Commits specification.

Commit messages MUST be formatted as:

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Example:

```text
feat(auth): add OAuth login support
```

The commit contains the following structural elements, to communicate intent to the consumers of your library:

1. `fix:` a commit of the type `fix` patches a bug in your codebase.
2. `feat:` a commit of the type `feat` introduces a new feature to the codebase.
3. `BREAKING CHANGE:` a footer or `!` after the type/scope introduces a breaking API change.
4. Additional types are allowed (for example: `docs:`, `refactor:`, `test:`, `chore:`).

Write commit messages in the imperative mood and keep descriptions concise and specific.

Reference:
https://www.conventionalcommits.org/en/v1.0.0/#summary

---

## Changelog Guidelines

We maintain `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

Before opening a pull request, add an entry under `## [Unreleased]` for any user-facing changes:

- **End-User Friendly**: Make entries easy to understand for end users. For new features, explain what they actually do in plain language without heavy technical jargon. For bug fixes, describe what the bug was in a friendly, relatable way rather than focusing on internal code mechanics, file paths, or private symbols.
- **Formatting**: `- **<scope>**: <Description>` (e.g. `- **asset**: Fix an issue where moving a video to trash while it was still transcoding could cause it to lose its trashed status`).
- **No PR Numbers**: Do not include PR numbers or links; the release pipeline automatically appends the detailed PR list.
- **Internal Changes**: Routine refactors, test additions, or internal chores without external behavior changes may omit changelog updates.

---

## Pull Request Template

```md
## Summary

Briefly describe what this PR changes and why.

## Changes

- List the main updates made in this branch.
- Include any important implementation details.

## Verification

- Describe the checks or tests you ran.
- Include relevant outputs if applicable.

## Notes

Add any additional context, caveats, or follow-up work.
```

## Submission Rules

- **Strict Requirement**: A submission is considered complete **only** when there is a single final code state in which **all** of the following pass **simultaneously**:
  - `bun run lint`
  - `bun run format`
  - `bun run typecheck`
  - `bun run test`
  - `bun run test:e2e:app`
  - `bun run test:e2e:webui`
  - `bun run test:e2e:workflow`

- **Backend Testing Mandate**: Every backend feature, service method, workflow, and activity MUST be accompanied by comprehensive tests. Logic-heavy code without corresponding test coverage is considered incomplete.

- Fixes must be iterated until **no check causes any other check to fail**.

- **Type Safety**: The use of explicit `any` is strictly forbidden and will result in lint errors. You should use `unknown` instead when the type is not known. If you absolutely must use `any` due to a limitation (e.g. interacting with an untyped 3rd party library), you must add an eslint-disable comment (e.g., `// eslint-disable-next-line @typescript-eslint/no-explicit-any`) and include a comment directly above it clearly explaining _why_ we cannot be type-safe here.
- Do **not** submit intermediate states where some checks pass and others fail, even temporarily.

- **Cleanup Requirement**: Remove all verification related files (scripts, screenshots, `verification/` folder) before submit.

## Backend Architecture

The backend is built with:

- **Runtime**: Bun
- **Framework**: Hono
- **ORM**: Prisma (with Pgvector18)
- **Database**: Pgvector18

## Workspace Architecture

The project is a monorepo managed by **Bun Workspaces**. It follows a strictly decoupled architecture where each domain or layer is its own package:

- **WebUI (`packages/webui`)**: React-based frontend.
- **API (`packages/api`)**: Hono-based HTTP entry point. Handles requests and calls Core services.
- **Core (`packages/core`)**: Business logic, services, and infrastructure utilities.
- **Database (`packages/db`)**: Prisma client, schema, and migrations.
- **DTOs (`packages/dtos`)**: Shared type definitions and Zod schemas used by both API and WebUI.
- **Workers**: Specialized packages for background task execution:
  - `@shumai/workflow-core`: Common workflow engine logic.
  - `@shumai/agent`: AI agent workflows and activities.
  - `@shumai/transcode`: Media processing workflows and activities.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shumaiOne/shumai](https://github.com/shumaiOne/shumai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->

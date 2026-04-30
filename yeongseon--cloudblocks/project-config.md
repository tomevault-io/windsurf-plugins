---
trigger: always_on
description: `cloudblocks` is a monorepo for a visual cloud architecture builder with a TypeScript frontend and a scaffolded Python backend.
---

# AGENTS.md

## Purpose

`cloudblocks` is a monorepo for a visual cloud architecture builder with a TypeScript frontend and a scaffolded Python backend.

## Read First

- `README.md`
- `CONTRIBUTING.md`

## Working Rules

- Preserve the existing visual-model vocabulary: blocks, connections, templates.
- Keep frontend behavior, docs, and any domain model changes synchronized.
- Treat `apps/web` as the primary production surface unless the task explicitly targets `apps/api`.
- Avoid incidental refactors in areas that already have unrelated user changes.
- **UNIVERSAL PORT STANDARD (INVIOLABLE)**: Every port in the system — container blocks, resource blocks, any element — uses identical dimensions: rx=12, ry=6, height=5, 3-layer structure (shadow + top + inner ring). Only colors vary. This is the block-based composition principle: same gauge = assembly possible. See `docs/design/BRICK_DESIGN_SPEC.md §0` (historical reference). Any change that produces non-uniform ports is a blocking defect.
- **English only**: All documentation, code comments, UI strings, and commit messages must be written in English. Do not introduce Korean or any other non-English text. An i18n system (`react-i18next`) is planned for future localization — until then, English is the single source language.
- **Historical documents are immutable**: Do NOT edit documents marked "Historical (Superseded)" — including `BRICK_DESIGN_SPEC.md`, `VISUAL_DESIGN_SPEC.md`, and `BRICK_GUIDEBOOK.md`. ADRs (`docs/adr/`) are also immutable once merged; create a new ADR to supersede an old one.
- **SVG asset rules**: All SVG sprites live in `apps/web/src/shared/assets/`. New SVG files must comply with the Universal Port Standard. Use consistent naming: lowercase kebab-case (e.g., `internet.svg`, `compute-block.svg`). Every SVG must include a `viewBox` attribute and avoid inline `style` elements — use attributes or CSS classes instead.
- **Zustand store boundaries**: Three stores exist — `architectureStore` (domain model: container blocks, resource blocks, connections, external actors), `uiStore` (UI state: tool mode, panel visibility, selection), and `authStore` (auth: GitHub OAuth, session). Add new state to the store that owns the domain. Do not create new stores without discussion.
- **Test expectations**: New features should include tests. Branch coverage must stay ≥ 90%. Do not delete or skip failing tests to make CI pass — fix the root cause instead.

## Git Conventions

### Branch Naming

Use the pattern `{type}/{issue#}-{short-desc}` or `{type}/{short-desc}` for branches:

- `feat/447-menubar-consolidation`
- `fix/441-external-actor-css`
- `docs/readme-badges`
- `chore/release-management`
- `m14/ai-frontend-integration` (milestone-scoped work)

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/). Format: `{type}({scope}): {description}`.

| Prefix     | When to use                             |
| ---------- | --------------------------------------- |
| `feat`     | New feature or capability               |
| `fix`      | Bug fix                                 |
| `docs`     | Documentation only                      |
| `test`     | Adding or updating tests                |
| `refactor` | Code change that neither fixes nor adds |
| `style`    | Formatting, CSS, whitespace             |
| `chore`    | Build, tooling, release management      |
| `perf`     | Performance improvement                 |
| `ci`       | CI/CD pipeline changes                  |

Scope is optional but recommended: `feat(web):`, `fix(api):`, `docs:`.

### Pull Request Rules

- `main` is a protected branch — all changes go through PR + CI.
- Always request Copilot code review: include `--reviewer copilot-pull-request-reviewer` in every `gh pr create` command. Auto-review is enabled at the repo level but may not trigger reliably — the explicit flag guarantees it.
- Squash-merge every PR with `--delete-branch`: `gh pr merge <number> --squash --delete-branch`. **Never** use `--admin` — let CI gates enforce quality.
- PR title should follow the same Conventional Commits format as commit messages.
- Each PR should reference and close its issue (e.g., `Fixes #123`).
- **Post-PR review gate**: After creating or updating a PR, always check GitHub's automated review results before merging:
  1. Wait for CI checks to complete: `gh pr checks <number> --watch`
  2. Review any automated review comments: `gh api repos/{owner}/{repo}/pulls/<number>/reviews` and `gh api repos/{owner}/{repo}/pulls/<number>/comments`
  3. If automated reviewers flag issues, fix them on the branch and push before merging.
  4. Only proceed with merge when all checks pass and no unresolved review comments remain.

## Implementation Principles

### Use Proven Libraries First

**Don't reinvent the wheel.** Before implementing any feature from scratch, search for well-maintained libraries that solve the problem.

| Need            | Prefer                          | Avoid                               |
| --------------- | ------------------------------- | ----------------------------------- |
| Drag & drop     | `interactjs`, `@dnd-kit`        | Custom pointer event handling       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yeongseon/cloudblocks](https://github.com/yeongseon/cloudblocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

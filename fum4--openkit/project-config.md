---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Mirror File Requirement

- `CLAUDE.md` and `AGENTS.md` must stay in sync.
- Any change made to one must be applied to the other in the same update.

## Naming and File Hygiene

- Choose clear, specific names for files and folders.
- When changing behavior or purpose, evaluate whether the file/folder name is still accurate.
- If the current name no longer fits, rename it as part of the change.
- Be careful when editing file/folder content to keep structure and naming aligned.

## README Requirement

Every app (`apps/*`) and library (`libs/*`) must have a `README.md` at its root. The README should briefly describe the package's purpose, key technologies, and basic usage. When creating a new app or library, include a README as part of the initial setup.

## CLI-First Agent Design

**Design new deterministic capabilities in the CLI first, so agents can automate workflows without UI coupling.**

- Prefer adding explicit CLI commands/flags over hidden behavior in prompts or UI-only flows.
- If a workflow has deterministic decisions (resolution, validation, initialization), expose them as CLI commands.
- Keep command output machine-readable where useful (for example JSON modes).
- Agent instructions should primarily orchestrate the CLI, not duplicate business logic.

## Code Layering

**The server app is an orchestration layer only.** It must not contain business logic, git operations, or third-party integration code directly. All primitives live in libraries and are composed together in the server routes like building blocks.

- **`libs/shared`** — All business logic, system operations, pure git primitives (stage, unstage, revert, diff, commit, push, status), types, and helpers.
- **`libs/integrations`** — All third-party provider integrations (GitHub CLI auth/PRs, Jira, Linear). Depends on `libs/shared`, never the other way around.
- **`apps/server/routes`** — Thin HTTP handlers that import from `libs/shared` and `libs/integrations`, validate request params, and return responses. No inline `execFile` calls, no embedded business logic.

When adding a new git operation or integration feature, implement it in the appropriate library first, then wire it up through a thin route handler.

## Quick Reference

**Package manager**: pnpm

**Test runner**: Vitest. Run `pnpm test:unit` for all, `pnpm nx run <project>:test:unit` for one project.

**Always use scripts from the root `package.json`** when available (`pnpm test:unit`, `pnpm check:*`, `pnpm fix:*`, `pnpm smoke`, etc.). Never invoke internal tools directly via `npx`, bare binary names, or raw `vitest`/`oxlint`/`oxfmt` commands — the `package.json` scripts ensure correct configuration, project boundaries, and environment setup. Running tools directly can pick up files from worktrees, skip required configs, or use wrong versions.

## Testing

**Tests are the most important aspect of this codebase.** They are the safety net for refactoring, bug prevention, and CI/CD readiness. Every code change — feature, bugfix, or refactor — must include corresponding tests. Writing tests is not optional or secondary; it is the primary deliverable alongside working code.

- When writing or modifying tests, **always use the testing skill** (`.claude/skills/testing/`) if available. It contains the canonical patterns, query priorities, and conventions for this project.
- **Do not modify existing tests lightly.** If a test fails, first verify whether the test caught a real bug before changing it. Changing a test to make it pass defeats the purpose — investigate first.
- Put real effort into test quality. Tests should be thorough, covering edge cases, error paths, and boundary conditions — not just the happy path.
- **Place test files in `__test__/` directories**, not co-located with source files. Each directory level that contains testable code should have its own `__test__/` subdirectory (e.g., `src/__test__/`, `src/adapters/__test__/`). Test files keep the `.test.ts`/`.test.tsx` suffix.
- Write tests carefully — one behavior per `it()`, Arrange-Act-Assert structure, behavior-spec naming.
- Mock at the boundary (fs, child_process, HTTP), not internal helpers.
- Component tests use React Testing Library: query by role/label/text, use `userEvent`, never test implementation details.
- **Never use `title` attributes in tests** (e.g., `getByTitle`). The `title` attribute renders native browser tooltips which conflict with the app's custom `Tooltip` component. Instead, query by displayed text (`getByText`), by role (`getByRole("button", { name: "..." })`), or by `aria-label`.
- **Every codebase change must have test coverage.** When creating or modifying code, always create or update the corresponding unit tests. No exceptions.

## Code Quality

**Fix any lint or format errors you encounter — whether introduced by current changes or pre-existing in the codebase.** Don't leave broken windows.

- Never use Prettier in this repository.
- Use `oxlint` for linting and `oxfmt` for formatting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fum4/openkit](https://github.com/fum4/openkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

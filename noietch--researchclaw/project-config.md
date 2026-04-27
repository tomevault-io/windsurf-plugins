---
trigger: always_on
description: This file defines default engineering constraints for all changes in this repository.
---

# Repository Rules for Claude Code

This file defines default engineering constraints for all changes in this repository.

## Project Overview

**ResearchClaw** is a standalone Electron desktop app for researchers. It is NOT a Claude Code plugin.

```
src/
  main/       # Electron main process (IPC handlers, services, stores)
  renderer/   # Vite + React UI
  shared/     # Shared types, utils, prompts (no Node/Electron deps)
  db/         # Prisma client + repositories
prisma/       # schema.prisma
tests/        # Integration tests (service layer, no Electron needed)
scripts/      # build-main.mjs, build-release.sh
```

**Path aliases** (tsconfig + esbuild + vite + vitest):

- `@shared` → `src/shared/index.ts`
- `@db` → `src/db/index.ts`
- `@/*` → `src/renderer/*`

## Process Management

- **Never run `pkill -f electron` or `killall electron`** — this will kill ALL Electron-based apps on the system (VS Code, Cursor, etc.), not just ResearchClaw.
- To stop ResearchClaw specifically, always use: `pkill -f ResearchClaw`

## PR Submit

- You should use the atomic-commit-reorganize SKILLS to reorganize the commit before you pull a PR.

## Scope and Priority

- Scope: Entire repository (`researchclaw`).
- Priority: These rules are the default for every implementation unless the user gives explicit one-off overrides in a task.

## Mandatory standards

1. **Tests must cover real business chain**
   - Must include realistic sample import flow (Chrome history sample data).
   - Must cover paper -> reading card workflow.
   - Health/help-only tests are insufficient for feature acceptance.

2. **Every coding session must update `changelog.md`**
   - Append a concise entry under the relevant version/date section in `changelog.md` (root of repo).
   - Entry must summarize what changed and affected scope. No separate per-file entries needed.

3. **Formatting + lint/review checks must pass before commit**
   - Pre-commit checks are required.
   - **Always run `npm run lint` before every commit** to ensure code formatting is correct.
   - If lint fails, run `npx prettier . --write` to auto-fix formatting issues.
   - **Always run `npm run test` before every commit** to ensure all tests pass.
   - Tests that require API keys should use `requiresModelIt` helper to skip in CI environments.

4. **Database schema changes require migration**
   - When adding new features that modify `prisma/schema.prisma`, always run `npx prisma db push` to sync the database.
   - Remind user to run migration if schema changes are detected.
   - **Note**: Database path is `{RESEARCH_CLAW_STORAGE_DIR}/researchclaw.db` (defaults to `~/.researchclaw/researchclaw.db`). Update `.env` DATABASE_URL accordingly before running CLI commands.

5. **Commit working code immediately**
   - When a feature is functionally complete and passes type checks, commit it right away.
   - Use `git add` and `git commit` promptly to preserve work.
   - This prevents accidental loss of code from git operations (checkout, reset, etc.).
   - Commit message format: `feat/fix/refactor: brief description`

6. **Only commit and push files you modified**
   - Always use `git add <specific files>` — never `git add .` or `git add -A`.
   - Do not stage or push files you did not touch, even if they appear in `git status`.

7. **README must be updated in both Chinese and English**
   - `README.md` (English) and `README_CN.md` (Chinese) are separate files.
   - When updating README, always update both files to keep them synchronized.

8. **Branch and PR workflow**
   - **Main branch (`main`) is protected and must only be updated via Pull Requests.**
   - Never push directly to `main` branch.
   - All feature development must be done in feature branches (e.g., `feat/feature-name`, `fix/bug-name`).
   - When feature work is complete, create a PR to merge into `main`.
   - Feature branches can be pushed directly for collaboration and backup.

9. **Long-running operations must use background jobs**
   - Any IPC operation that may take more than a few seconds (LLM streaming, comparison, recommendation generation, etc.) must run as a background job in the main process.
   - The main process keeps job state in memory and broadcasts progress via `BrowserWindow.webContents.send`.
   - The renderer must **never** auto-kill a background job on component unmount — users should be able to navigate away and return to see results.
   - Provide a `getActiveJobs` / status query handler so the renderer can recover in-progress or completed job state when the page remounts.
   - Users may manually cancel a job via an explicit Stop/Cancel button.

## Expected coding sequence

1. Create changelog entry for the coding session.
2. Implement feature and tests.
3. Fill changelog test design + validation results.
4. Run formatting/lint/test checks.
5. Commit only when checks pass.

## UI Design Language

Notion-inspired design: clean whites, soft grays, light blue accents, smooth micro-interactions.

### 1. Color System

All colors use `notion-*` Tailwind tokens:

- **Backgrounds**: `bg-white` (cards) · `bg-notion-sidebar` #f7f7f5 (sidebar) · `bg-notion-accent-light` #e8f4f8 (hover/selected)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noietch/ResearchClaw](https://github.com/Noietch/ResearchClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

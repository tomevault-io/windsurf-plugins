---
trigger: always_on
description: > Rules for AI coding agents working in this repository.
---

# GitNotēs — Agent Rules

> Rules for AI coding agents working in this repository.

## Read the Wiki First

**Before touching any code, read the relevant wiki page(s).** The wiki (`docs/wiki/`) is the canonical reference for every service, store, hook, model, screen, context, and architectural pattern. If you need to understand how something works, the wiki has the answer.

- Changing a service? → read `docs/wiki/services.md`
- Modifying state? → read `docs/wiki/stores.md`
- Editing screens or navigation? → read `docs/wiki/screens.md`
- Touching sync or Git logic? → read `docs/wiki/sync-architecture.md`
- Working with notes? → read `docs/wiki/note-file-format.md`
- Anything involving the native Git module? → read `docs/wiki/git-engine.md`

If the wiki is wrong or missing, fix it first — then make your code change. See the wiki accuracy rule below.

## Worktrees (ALWAYS)

**All agent work — fixes, features, refactors, even small edits — must happen inside a git worktree. Never edit files in the main worktree's working tree directly.**

This repo runs **multiple concurrent agent sessions** on different branches and issues. The main worktree is a shared resource — all agents serve from it and commit against it. Editing it directly causes race conditions where one agent's uncommitted changes block or corrupt another's work. Use worktrees to isolate every session's changes.

### Workflow

```bash
# From the repo root (the main worktree):
git worktree add -b <type>/<scope>-<slug> .worktrees/<scope>-<slug> main
# e.g.  git worktree add -b fix/discard-placeholder .worktrees/fix-discard main

# Always create the worktree under .worktrees/ at the repo root — never in
# /tmp, the home directory, or anywhere outside the repo. .worktrees/ is
# gitignored so the symlink/node_modules and per-worktree state don't pollute git.

ln -sfn "$(pwd)/node_modules" .worktrees/<scope>-<slug>/node_modules
# node_modules is a 2.5 GB tree that is symlinked from main into the worktree
# so jest (which resolves from the worktree) works without a duplicate install.
```

### Rules

- **Never leave code in main.** The main working tree must always be clean — no uncommitted changes, no staged files, no dangling work. If your worktree branch is merged, remove the worktree immediately. A dirty main blocks other agents and causes race conditions.
- **One worktree per branch.** Never branch from another worktree's branch — always branch from `main` (or the upstream you're targeting). After `git fetch origin` in the main repo, base new worktrees on the updated `origin/main`.
- **Coordinate before touching shared files.** Before editing `conflictStore.ts`, `LocalGitWriter.ts`, or any file another agent's `git status` shows as modified in the main working tree, check `git worktree list` and `git status` in the other worktrees. If another session has uncommitted work on the same files, wait or scope your change to a different file.
- **Do not commit secrets / tokens / Metro debug output** — review `git diff` before committing. This applies whether you are in a worktree or not.
- **Clean up** with `git worktree remove <path>` when a branch is merged and the worktree is no longer needed. Branches are cheap to recreate.
- **Main must stay clean.** After merging (squash-merge or rebase+merge), immediately remove the source worktree and verify `git status` in main shows nothing uncommitted or unstaged. A polluted main blocks other agents and corrupts shared state.

## Testing

All tests must pass before pushing:

```bash
yarn ts:check       # TypeScript compilation
yarn jest           # Run all Jest tests
yarn eslint . --ext .ts,.tsx  # Linting
```

- **Never push failing tests.** CI runs the same suite.
- New service/hook/feature needs a test file in `__tests__/` mirroring the `src/` structure.
- Use `@testing-library/react-native` for component tests.
- Use `jest.mock()` for service dependencies, `Date.now()` mocking for cache tests.
- Run `yarn jest __tests__/specific-file.test.ts --no-coverage --forceExit` for targeted testing.

## Self-Documenting Code

- Clear names, no comments explaining obvious behavior.
- Use TypeScript strict mode — no `any` without justification.
- Keep functions short and focused (single responsibility).
- Use the existing patterns in `src/services/` as reference.
- **Never keep deprecated code.** When a library/module deprecates an API (e.g., `FileSystem.deleteAsync`), migrate to the replacement immediately. Do not leave deprecated calls in the codebase — fix them as part of the same change that introduces them.
- **Zero tolerance for lint-blocking issues.** Before committing, verify `yarn eslint src --ext .ts,.tsx` reports **0 errors**. Agents must proactively eliminate these categories on every change:
  - `@typescript-eslint/no-unused-vars` — remove unused imports, variables, and function parameters; prefix intentionally unused parameters with `_`
  - `@typescript-eslint/no-empty-function` — replace empty function bodies with `/* noop */` comments (never leave `{}` or `async () => {}`)
  - `prefer-const` — change `let` to `const` when the variable is never reassigned
  - `no-useless-escape` — remove unnecessary escape characters in regex (`\[` → `[`, `\]` → `]`, `\-` → `-`, `\/` → `/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skepjandi/gitnotes](https://github.com/skepjandi/gitnotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

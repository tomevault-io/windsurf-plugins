---
trigger: always_on
description: When you encounter unexpected issues, workarounds, or non-obvious behavior (CI quirks, tooling gotchas, environment-specific problems), **update this AGENTS.md** with the finding so future sessions don't repeat the investigation. Add the note to the most relevant existing section, or create a new one if needed.
---

# Grackle Development Guidelines

## Self-Updating Documentation

When you encounter unexpected issues, workarounds, or non-obvious behavior (CI quirks, tooling gotchas, environment-specific problems), **update this AGENTS.md** with the finding so future sessions don't repeat the investigation. Add the note to the most relevant existing section, or create a new one if needed.

### Package READMEs

Each package under `packages/` has a `README.md` that is published to npm. **When you change a package's public behavior** (new commands, renamed options, new MCP tools, new API methods, changed configuration, etc.), **update that package's README to match.** Do not let READMEs drift out of sync with the code.

- **CLI** (`packages/cli/README.md`): documents every command, subcommand, and flag. Update when adding/removing/renaming commands or options.
- **MCP** (`packages/mcp/README.md`): documents every MCP tool with parameters. Update when adding/removing/renaming tools.
- **Knowledge Core** (`packages/knowledge-core/README.md`): detailed API reference. Update when the public API changes.
- **All other packages**: concise overviews. Update if the package's purpose, key concepts, or usage instructions change.

Do **not** update the root `README.md` as part of this process — it is a separate marketing/overview document maintained independently.

### Environment Variables

The root `.env.example` documents every supported environment variable with defaults and descriptions. **When you add, remove, or rename a `process.env` read**, update `.env.example` to match.

## Git Workflow

- **Never push directly to main.** All changes must go through a pull request. Always create a feature branch and open a PR — never commit and push to `main`.
- **Never rebase or force-push.** To sync with `main`, first run `git fetch origin` and then use `git merge origin/main` instead of `git rebase`. Rebasing published branches rewrites history and typically requires a force-push, which we do not allow.
- **Never merge PRs** unless the user explicitly tells you to merge. Other agents may be coordinating merge order.
- **Branch naming**: `<github-username>/<issue>-<feature>` when working on a GitHub issue (where `<issue>` is the numeric issue id, e.g., `nick-pape/149-agent-subtask-creation`), or `<github-username>/<feature>` when there's no issue (e.g., `nick-pape/fix-typo-in-readme`).

### Worktree Workflow

**All feature work must happen in a worktree**, not on the main working copy. Use standard git worktree commands to create an isolated worktree for each feature.

**Starting work:**
1. From the repository root, create a new worktree and branch with a descriptive name, for example: `git fetch origin && git worktree add -b <github-username>/123-my-feature ../grackle-123-my-feature origin/main`
2. Change into the new worktree directory: `cd ../grackle-123-my-feature`
3. Run `rush install && rush build`
4. Do all your work from that worktree so the branch, cwd, and hooks stay isolated from the main working copy

**Finishing work:**
- After the PR is merged and the worktree is no longer needed, return to the main repository checkout and remove it with `git worktree remove ../grackle-123-my-feature`
- To pause and come back later, leave the worktree in place and return to it when needed

**Rules:**
- Never check out a feature branch in the main working copy — always use a separate git worktree
- Name the branch according to our convention when creating it: `<github-username>/<issue>-<feature>` or `<github-username>/<feature>`
- Each worktree needs its own `rush install && rush build` (node_modules are per-worktree)
- You can't have the same branch checked out in two worktrees simultaneously
- When syncing with main inside a worktree: `git fetch origin && git merge origin/main` (same as always, no rebase)

## Planning

- **Always plan tests**: Every implementation plan must include a section for tests (E2E Playwright specs for `@grackle-ai/web`, unit/integration tests for other packages). If the change is purely cosmetic or untestable, explicitly note why tests are skipped.
- **Prefer `data-testid` in E2E tests**: Use `data-testid` attributes and `page.getByTestId()` to locate elements in Playwright tests rather than fragile DOM selectors like `getByText()` with `{ exact: true }`. Text-based locators break when the same text appears in multiple places (e.g., StatusBar and page content). Add `data-testid` to components when writing tests that need to disambiguate.
- **Scope workspace-name locators to the sidebar in E2E tests**: the dashboard home page reuses workspace names in cards, so unscoped `page.getByText(workspaceName)` calls can hit Playwright strict-mode violations. Prefer helpers or locators rooted under `data-testid="sidebar"` when clicking workspace rows or adjacent sidebar controls.
- **Open a PR as the final step**: Use `/create-pr` to open the PR. The PR body must link back to the issue.

## Build & Test


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nick-pape/grackle](https://github.com/nick-pape/grackle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->

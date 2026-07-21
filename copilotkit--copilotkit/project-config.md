---
trigger: always_on
description: <!-- nx configuration start-->
---

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- You have access to the Nx MCP server and its tools, use them to help the user
- When answering questions about the repository, use the `nx_workspace` tool first to gain an understanding of the workspace architecture where applicable.
- When working in individual projects, use the `nx_project_details` mcp tool to analyze and understand the specific project structure and dependencies
- For questions around nx configuration, best practices or if you're unsure, use the `nx_docs` tool to get relevant, up-to-date docs. Always use this instead of assuming things about nx configuration
- If the user needs help with an Nx configuration or project graph error, use the `nx_workspace` tool to get any errors
- For Nx plugin best practices, check `node_modules/@nx/<plugin>/PLUGIN.md`. Not all plugins have this file - proceed without it if unavailable.

<!-- nx configuration end-->

> Working under `showcase/`? Read `showcase/AGENTS.md` FIRST — it defines the non-negotiable iron rules for showcase cells.

# CopilotKit

AI agent framework with three layers: **Frontend** (React/Angular/Vanilla) → **Runtime** (Express/Hono) → **Agent** (LangGraph/CrewAI/BuiltIn/Custom), communicating via the AG-UI protocol (event-based SSE).

## Essentials

- **Nx monorepo** — always run tasks through `nx` (`nx run`, `nx run-many`, `nx affected`), never the underlying tooling directly.
- **Flat package structure** — All packages live under `packages/` with the `@copilotkit/` scope. Some packages have `v1/` and `v2/` internal directories for backward compatibility, but they're a single published package.
- **Simplicity** — prefer the simplest correct solution. For non-trivial changes, consider if there's a cleaner approach before committing.
- **Worktrees** — always work in a git worktree for isolation. See [Git & PRs](.claude/docs/git.md) for the full workflow.
- **Commit as you go** — every meaningful unit of work gets its own commit, pushed immediately. Don't let untracked files accumulate across a session. Tests belong in the commit that introduces the code being tested. Full rules in [Git & PRs](.claude/docs/git.md#commit-early-and-often-in-logical-chunks).
- **Draft PR up front** — the moment a new branch has one commit, push it and open a **draft PR**. Don't wait until "ready" — unmerged-and-unpushed work is invisible. Flip the PR from draft to ready (`gh pr ready <pr#>`) only when the developer says so. See [Git & PRs](.claude/docs/git.md#open-a-draft-pr-up-front).
- **Documentation lives in shell-docs** — author CopilotKit docs in `showcase/shell-docs/src/content/`. The top-level `docs/` path is only a symlink to `showcase/shell-docs/`; never recreate the old `docs/content/docs/` tree for live documentation. AG-UI protocol docs are authored upstream in `ag-ui-protocol/ag-ui`, not directly in this repo. See [Documentation](.claude/docs/documentation.md).

## Private Agent Instructions

Individual developers may optionally create a `private-agents.md` file at the repo root. This file is gitignored and not shared with the team -- it contains personal agent instructions, workflow overrides, or context that applies only to that developer's work. If `private-agents.md` exists, read it and follow its instructions (they take precedence over the defaults in this file where they conflict).

## Internal Skills

The team maintains shared AI agent skills at [CopilotKit/internal-skills](https://github.com/CopilotKit/internal-skills). If installed as a Claude Code plugin, these skills are available automatically. Key skills relevant to this repo:

- **copilotkit-ui-theme** — CopilotCloud visual design system (colors, typography, glass effects, blur circles). Use when building any UI that should look like an official CopilotKit product.
- **copilotkit-branding** — Brand rules, logos, and visual identity guidelines.
- **copilotkit-dev-workflow** — Internal dev workflow conventions for this monorepo.
- **cr-loop** — Automated code review and fix loop.

If you need a skill and don't have the plugin installed, clone the repo and read the relevant `skills/<name>/SKILL.md` directly.

## Documentation Editing

Before editing anything that looks like product docs, read [Documentation](.claude/docs/documentation.md) and the local README for the docs area you are touching. The live docs source is **`showcase/shell-docs/`**; top-level `docs/` is only a symlink there.

- **CopilotKit product docs** live under `showcase/shell-docs/src/content/`:
  - Guides, how-tos, and concepts: `showcase/shell-docs/src/content/docs/`
  - API reference: `showcase/shell-docs/src/content/reference/`
  - Shared MDX snippets: `showcase/shell-docs/src/content/snippets/`
  - Framework overview pages: `showcase/shell-docs/src/content/framework-overviews/`
- When adding or moving a guide page under `showcase/shell-docs/src/content/docs/`, update that section's `meta.json` so the page appears in navigation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CopilotKit/CopilotKit](https://github.com/CopilotKit/CopilotKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

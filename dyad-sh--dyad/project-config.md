---
trigger: always_on
description: Please read `CONTRIBUTING.md` which includes information for human code contributors. Much of the information is applicable to you as well.
---

# Repository Agent Guide

Please read `CONTRIBUTING.md` which includes information for human code contributors. Much of the information is applicable to you as well.

## Rules index

> **IMPORTANT: BEFORE writing any code or making changes, you MUST read the relevant rule files from the table below.** Identify which areas your task touches and read those rule files first. Skipping this step leads to avoidable mistakes and rework.

Detailed rules and learnings are in the `rules/` directory. Read the relevant file when working in that area.

| File                                                                 | Read when...                                                                                                                                                                   |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [rules/electron-ipc.md](rules/electron-ipc.md)                       | Adding/modifying IPC endpoints, handlers, React Query hooks, or renderer-to-main communication                                                                                 |
| [rules/dyad-errors.md](rules/dyad-errors.md)                         | Classifying IPC/main errors with `DyadError` / `DyadErrorKind` and PostHog exception filtering                                                                                 |
| [rules/local-agent-tools.md](rules/local-agent-tools.md)             | Adding/modifying local agent tools, tool flags (`modifiesState`), or read-only/plan-only guards                                                                                |
| [rules/e2e-testing.md](rules/e2e-testing.md)                         | Writing or debugging E2E tests (Playwright, Base UI radio clicks, Lexical editor, test fixtures)                                                                               |
| [rules/git-workflow.md](rules/git-workflow.md)                       | Pushing branches, creating PRs, or dealing with fork/upstream remotes                                                                                                          |
| [rules/base-ui-components.md](rules/base-ui-components.md)           | Using TooltipTrigger, ToggleGroupItem, or other Base UI wrapper components                                                                                                     |
| [rules/database-drizzle.md](rules/database-drizzle.md)               | Modifying the database schema, generating migrations, or resolving migration conflicts                                                                                         |
| [rules/native-modules.md](rules/native-modules.md)                   | Adding Electron native modules or binaries that must survive Forge packaging/rebuild                                                                                           |
| [rules/typescript-strict-mode.md](rules/typescript-strict-mode.md)   | Debugging type errors from `npm run ts` (tsgo) that pass normal tsc                                                                                                            |
| [rules/openai-reasoning-models.md](rules/openai-reasoning-models.md) | Working with OpenAI reasoning model (o1/o3/o4-mini) conversation history                                                                                                       |
| [rules/adding-settings.md](rules/adding-settings.md)                 | Adding a new user-facing setting or toggle to the Settings page                                                                                                                |
| [rules/chat-message-indicators.md](rules/chat-message-indicators.md) | Using `<dyad-status>` tags in chat messages for system indicators                                                                                                              |
| [rules/supabase-functions.md](rules/supabase-functions.md)           | Deploying, bundling, or queueing Supabase Edge Functions                                                                                                                       |
| [rules/product-principles.md](rules/product-principles.md)           | Planning new features, especially via `dyad:swarm-to-plan`, to guide design trade-offs                                                                                         |
| [rules/jotai-testing.md](rules/jotai-testing.md)                     | Unit-testing Jotai atoms/hooks with `renderHook`, especially across unmount/remount                                                                                            |
| [rules/claude-github-workflows.md](rules/claude-github-workflows.md) | Editing `.github/workflows/*.yml` that invoke `anthropics/claude-code-action` — workflow shape, untrusted-input handling, and **permission/`.claude/settings.json` hardening** |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dyad-sh/dyad](https://github.com/dyad-sh/dyad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->

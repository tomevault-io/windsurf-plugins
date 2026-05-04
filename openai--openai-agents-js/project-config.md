---
trigger: always_on
description: This guide helps new contributors get started with the OpenAI Agents JS monorepo. It covers repo structure, how to test your work, available utilities, file locations, and guidelines for commits and PRs.
---

# Contributor Guide

This guide helps new contributors get started with the OpenAI Agents JS monorepo. It covers repo structure, how to test your work, available utilities, file locations, and guidelines for commits and PRs.

**Location:** `AGENTS.md` at the repository root.

## Table of Contents

1.  [Policies & Mandatory Rules](#policies--mandatory-rules)
2.  [Project Structure Guide](#project-structure-guide)
3.  [Operation Guide](#operation-guide)

## Policies & Mandatory Rules

### Mandatory Skill Usage

#### `$code-change-verification`

Run `$code-change-verification` before marking work complete when changes affect runtime code, tests, or build/test behavior.

Run it when you change:

- `packages/`, `examples/`, `helpers/`, `scripts/`, or `integration-tests/`
- Root build/test config such as `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `tsconfig*.json`, `eslint.config.*`, or `vitest*.ts`

You can skip `$code-change-verification` for docs-only or repo-meta changes (for example, `docs/`, `.agents/`, `README.md`, `AGENTS.md`, `.github/`), unless a user explicitly asks to run the full verification stack.

#### `$changeset-validation`

When you change anything under `packages/` or touch `.changeset/`, use `$changeset-validation` to create and validate the changeset before you treat the code as final. Codex must ensure an appropriate changeset exists that covers every changed package, and run this skill alongside `$code-change-verification` ahead of handoff. When writing the changeset summary, use a Conventional Commit-style message (for example, `fix: ...` or `feat: ...`) so it can serve as a commit title.

#### `$openai-knowledge`

When working on OpenAI API or OpenAI platform integrations in this repo (Responses API, tools, streaming, Realtime API, auth, models, rate limits, MCP, Agents SDK/ChatGPT Apps SDK), use `$openai-knowledge` to pull authoritative docs via the OpenAI Developer Docs MCP server (and guide setup if it is not configured).

#### `$implementation-strategy`

Before changing runtime code, exported APIs, external configuration, persisted schemas, wire protocols, or other user-facing behavior, use `$implementation-strategy` to decide the compatibility boundary and implementation shape. Judge breaking changes against the latest release tag, not unreleased branch-local churn. Interfaces introduced or changed after the latest release tag may be rewritten without compatibility shims unless they already have a released or otherwise supported durable-state consumer, or the user explicitly asks for a migration path.

#### `$pr-draft-summary`

When a task in this repo finishes with moderate-or-larger code changes, invoke `$pr-draft-summary` in the final handoff to generate the required PR summary block, branch suggestion, title, and draft description. Treat this as the default close-out step after runtime code, tests, examples, build/test configuration, or docs with behavior impact are changed.

Skip `$pr-draft-summary` only for trivial or conversation-only tasks, repo-meta/doc-only tasks without behavior impact, or when the user explicitly says not to include the PR draft block.

### ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in PLANS.md) from design to implementation. Store each ExecPlan file under `plans/` with a descriptive name, and create the directory if it does not exist. Call out compatibility risk only when the plan changes behavior shipped in the latest release tag or a released/otherwise supported durable format. Do not treat branch-local interface churn or unreleased post-tag changes on `main` as breaking by default; prefer direct replacement over compatibility layers in those cases. Confirm the approach when changes could impact package consumers or durable external data that is already supported outside the current branch.

## Project Structure Guide

### Overview

The OpenAI Agents JS repository is a pnpm-managed monorepo that provides:

- `packages/agents`: A convenience bundle exporting core and OpenAI packages.
- `packages/agents-core`: Core abstractions and runtime for agent workflows.
- `packages/agents-openai`: OpenAI-specific bindings and implementations.
- `packages/agents-realtime`: Realtime bindings and implementations.
- `packages/agents-extensions`: Extensions for agent workflows.
- `docs`: Documentation site powered by Astro.
- `examples`: Sample projects demonstrating usage patterns.
- `scripts`: Automation scripts (`dev.mts`, `embedMeta.ts`).
- `helpers`: Shared utilities for testing and other internal use.

### Repo Structure & Important Files

- `packages/agents-core/`, `packages/agents-openai/`, `packages/agents-realtime/`, `packages/agents-extensions/`: Each has its own `package.json`, `src/`, `test/`, and build scripts.
- `docs/`: Documentation source; develop with `pnpm docs:dev` or build with `pnpm docs:build`. Translated docs under `docs/src/content/docs/ja`, `docs/src/content/docs/ko`, and `docs/src/content/docs/zh` are generated via `pnpm docs:translate`; do not edit them manually.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openai/openai-agents-js](https://github.com/openai/openai-agents-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

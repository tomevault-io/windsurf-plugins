---
trigger: always_on
description: This guide helps new contributors get started with the OpenAI Agents Python repository. It covers repo structure, how to test your work, available utilities, and guidelines for commits and PRs.
---

# Contributor Guide

This guide helps new contributors get started with the OpenAI Agents Python repository. It covers repo structure, how to test your work, available utilities, and guidelines for commits and PRs.

**Location:** `AGENTS.md` at the repository root.

## Table of Contents

1. [Policies & Mandatory Rules](#policies--mandatory-rules)
2. [Project Structure Guide](#project-structure-guide)
3. [Operation Guide](#operation-guide)

## Policies & Mandatory Rules

### Mandatory Skill Usage

#### `$code-change-verification`

Run `$code-change-verification` before marking work complete when changes affect runtime code, tests, or build/test behavior.

Run it when you change:
- `src/agents/` (library code) or shared utilities.
- `tests/` or add or modify snapshot tests.
- `examples/`.
- Build or test configuration such as `pyproject.toml`, `Makefile`, `mkdocs.yml`, `docs/scripts/`, or CI workflows.

You can skip `$code-change-verification` for docs-only or repo-meta changes (for example, `docs/`, `.agents/`, `README.md`, `AGENTS.md`, `.github/`), unless a user explicitly asks to run the full verification stack.

#### `$openai-knowledge`

When working on OpenAI API or OpenAI platform integrations in this repo (Responses API, tools, streaming, Realtime API, auth, models, rate limits, MCP, Agents SDK or ChatGPT Apps SDK), use `$openai-knowledge` to pull authoritative docs via the OpenAI Developer Docs MCP server (and guide setup if it is not configured).

#### `$implementation-strategy`

Before changing runtime code, exported APIs, external configuration, persisted schemas, wire protocols, or other user-facing behavior, use `$implementation-strategy` to decide the compatibility boundary and implementation shape. Judge breaking changes against the latest release tag, not unreleased branch-local churn. Interfaces introduced or changed after the latest release tag may be rewritten without compatibility shims unless they define a released or explicitly supported durable external state boundary, or the user explicitly asks for a migration path. Unreleased persisted formats on `main` may be renumbered or squashed before release when intermediate snapshots are intentionally unsupported.

#### `$pr-draft-summary`

When a task in this repo finishes with moderate-or-larger code changes, invoke `$pr-draft-summary` in the final handoff to generate the required PR summary block, branch suggestion, title, and draft description. Treat this as the default close-out step after runtime code, tests, examples, build/test configuration, or docs with behavior impact are changed.

Skip `$pr-draft-summary` only for trivial or conversation-only tasks, repo-meta/doc-only tasks without behavior impact, or when the user explicitly says not to include the PR draft block.

### ExecPlans

Call out compatibility risk early in your plan only when the change affects behavior shipped in the latest release tag or a released or explicitly supported durable external state boundary, and confirm the approach before implementing changes that could impact users.

Use an ExecPlan when work is multi-step, spans several files, involves new features or refactors, or is likely to take more than about an hour. Start with the template and rules in `PLANS.md`, keep milestones and living sections (Progress, Surprises & Discoveries, Decision Log, Outcomes & Retrospective) up to date as you execute, and rewrite the plan if scope shifts. Call out compatibility risk only when the plan changes behavior shipped in the latest release tag or a released or explicitly supported durable external state boundary. Do not treat branch-local interface churn or unreleased post-tag changes on `main` as breaking by default; prefer direct replacement over compatibility layers in those cases, and renumber or squash unreleased persisted schemas before release when the intermediate snapshots are intentionally unsupported. If you intentionally skip an ExecPlan for a complex task, note why in your response so reviewers understand the choice.

### Public API Positional Compatibility

Treat the parameter and dataclass field order of exported runtime APIs as a compatibility contract.

- For public constructors (for example `RunConfig`, `FunctionTool`, `AgentHookContext`), preserve existing positional argument meaning. Do not insert new constructor parameters or dataclass fields in the middle of existing public order.
- When adding a new optional public field/parameter, append it to the end whenever possible and keep old fields in the same order.
- If reordering is unavoidable, add an explicit compatibility layer and regression tests that exercise the old positional call pattern.
- Prefer keyword arguments at call sites to reduce accidental breakage, but do not rely on this to justify breaking positional compatibility for public APIs.

### Platform, Docs, and Security Review

- Documentation is published to the live site, so coordinate SDK behavior changes and docs carefully. If docs describe behavior that is not released yet, either delay the docs change until the SDK release is available or split it into a follow-up PR.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openai/openai-agents-python](https://github.com/openai/openai-agents-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->

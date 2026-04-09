---
trigger: always_on
description: **Remote Agentic Coding Platform**: Control AI coding assistants (Claude Code SDK, Codex SDK) remotely from Slack, Telegram, and GitHub. Built with **Bun + TypeScript + SQLite/PostgreSQL**, single-developer tool for AI-assisted development practitioners. Architecture prioritizes simplicity, flexibility, and user control.
---

## Project Overview

**Remote Agentic Coding Platform**: Control AI coding assistants (Claude Code SDK, Codex SDK) remotely from Slack, Telegram, and GitHub. Built with **Bun + TypeScript + SQLite/PostgreSQL**, single-developer tool for AI-assisted development practitioners. Architecture prioritizes simplicity, flexibility, and user control.

## Core Principles

**Single-Developer Tool**
- No multi-tenant complexity

**Platform Agnostic**
- Unified conversation interface across Slack/Telegram/GitHub/cli/web
- Platform adapters implement `IPlatformAdapter`
- Stream/batch AI responses in real-time to all platforms

**Type Safety (CRITICAL)**
- Strict TypeScript configuration enforced
- All functions must have complete type annotations
- No `any` types without explicit justification
- Interfaces for all major abstractions

**Zod Schema Conventions**
- Schema naming: camelCase, descriptive suffix (e.g., `workflowRunSchema`, `errorSchema`)
- Type derivation: always use `z.infer<typeof schema>` — never write parallel hand-crafted interfaces
- Import `z` from `@hono/zod-openapi` (not from `zod` directly)
- All new/modified API routes must use `registerOpenApiRoute(createRoute({...}), handler)` — the local wrapper handles the TypedResponse bypass
- Route schemas live in `packages/server/src/routes/schemas/` — one file per domain
- Engine schemas live in `packages/workflows/src/schemas/` — one file per concern (dag-node, workflow, workflow-run, retry, loop, hooks); `index.ts` re-exports all
- Engine schema naming: camelCase (e.g., `dagNodeSchema`, `workflowBaseSchema`, `nodeOutputSchema`)
- `TRIGGER_RULES` and `WORKFLOW_HOOK_EVENTS` are derived from schema `.options` — never duplicate as a plain array (exception: `@archon/web` must define a local constant since `api.generated.d.ts` is type-only and cannot export runtime values)
- `loader.ts` uses `dagNodeSchema.safeParse()` for node validation; graph-level checks (cycles, deps, `$nodeId.output` refs) remain as imperative code in `validateDagStructure()`

**Git Workflow and Releases**
- `main` is the release branch. Never commit directly to `main`.
- `dev` is the working branch. All feature work branches off `dev` and merges back into `dev`.
- To release, use the `/release` skill. It compares `dev` to `main`, generates changelog entries, bumps the version, and creates a PR to merge `dev` into `main`.
- Releases follow Semantic Versioning: `/release` (patch), `/release minor`, `/release major`.
- Changelog lives in `CHANGELOG.md` and follows Keep a Changelog format.
- Version is the single `version` field in the root `package.json`.

**Git as First-Class Citizen**
- Let git handle what git does best (conflicts, uncommitted changes, branch management)
- Surface git errors to users for actionable issues (conflicts, uncommitted changes)
- Handle expected failure cases gracefully (missing directories during cleanup)
- Trust git's natural guardrails (e.g., refuse to remove worktree with uncommitted changes)
- Use `@archon/git` functions for git operations; use `execFileAsync` (not `exec`) when calling git directly
- Worktrees enable parallel development per conversation without branch conflicts
- Workspaces automatically sync with origin before worktree creation (ensures latest code)
- **NEVER run `git clean -fd`** - it permanently deletes untracked files (use `git checkout .` instead)

## Engineering Principles

These are implementation constraints, not slogans. Apply them by default.

**KISS — Keep It Simple, Stupid**
- Prefer straightforward control flow over clever meta-programming
- Prefer explicit branches and typed interfaces over hidden dynamic behavior
- Keep error paths obvious and localized

**YAGNI — You Aren't Gonna Need It**
- Do not add config keys, interface methods, feature flags, or workflow branches without a concrete accepted use case
- Do not introduce speculative abstractions without at least one current caller
- Keep unsupported paths explicit (error out) rather than adding partial fake support

**DRY + Rule of Three**
- Duplicate small, local logic when it preserves clarity
- Extract shared utilities only after the same pattern appears at least three times and has stabilized
- When extracting, preserve module boundaries and avoid hidden coupling

**SRP + ISP — Single Responsibility + Interface Segregation**
- Keep each module and package focused on one concern
- Extend behavior by implementing existing narrow interfaces (`IPlatformAdapter`, `IAssistantClient`, `IDatabase`, `IWorkflowStore`) whenever possible
- Avoid fat interfaces and "god modules" that mix policy, transport, and storage
- Do not add unrelated methods to an existing interface — define a new one

**Fail Fast + Explicit Errors** — Silent fallback in agent runtimes can create unsafe or costly behavior
- Prefer throwing early with a clear error for unsupported or unsafe states — never silently swallow errors
- Never silently broaden permissions or capabilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coleam00) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
